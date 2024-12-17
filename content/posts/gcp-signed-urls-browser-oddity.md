---
title: "GCP's Signed URLs & A Browser Oddity"
date: 2024-01-09
draft: false
tags: ["GCP", "Go", "JavaScript", "Cloud Storage"]
---

![Photo by Nuno Patricio taken from Flickr](/google-cloud.jpg)

A few weeks back I had to create a new GraphQL route in an existing Go application to handle simple file uploads to a GCP bucket for a project that I was building. Seemed pretty straightforward at first but a combination of misleading documentation and an unbeknown browser particularity made it way more convoluted than it needed to be.

First let's import the google cloud storage SDK for Go:

```go
import (
    "cloud.google.com/go/storage"
)
```

Then, to keep things clean, instead of injecting service account keys into the code, even for testing or development purposes, it's better to install the gcloud CLI tools and authenticate there with the service account that has the correct permissions to create objects:

```bash
$ gcloud auth login
```

Once logged in, I can then impersonate the correct service account:

```bash
$ gcloud auth application-default login --impersonate-service-account SERVICE_ACCT_EMAIL
```

This action will set the local application default credentials (ADC) to that of the service account. Now we can neatly use the service account's credentials without embedding them directly into the codebase. Additionally, this ensures seamless transition into production, as GCP expects code that operates with application default credentials (ADC).

Once this is set up, we can create the function that generates the signed URL:

```go
func GenerateSignedURL(objectName string, contentType string) (string, error) {
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    client, err := storage.NewClient(ctx)
    if err != nil {
        return "", fmt.Errorf("client connection error: %w", err)
    }
    defer client.Close()
    bucket := client.Bucket(bucketName)
    expirationTime := time.Now().Add(90 * time.Second)
    url, err := bucket.SignedURL(objectName, &storage.SignedURLOptions{
        Expires:     expirationTime,
        Method:      method,
        ContentType: contentType,
    })
    if err != nil {
        return "", fmt.Errorf("failed to generate signed URL: %w", err)
    }
    return url, nil
}
```

The code above is pretty basic; nothing inherently interesting is happening. The only thing that deserves attention is this part:

```go
url, err := bucket.SignedURL(objectName, &storage.SignedURLOptions{
    Expires:     expirationTime,
    Method:      method,
})
```

The method is provided by Google's storage SDK, and it takes two arguments: the name of the object and the options for generating the signed URL. As you can see, the only things I have put at the moment are the expiration time and the HTTP method, as these are the only things that are marked as required by the commented documentation of the SignedURLOptions object.

```go
 // Method is the HTTP method to be used with the signed URL.
 // Signed URLs can be used with GET, HEAD, PUT, and DELETE requests.
 // Required.
 Method string

 // Expires is the expiration time on the signed URL. It must be
 // a datetime in the future. For SigningSchemeV4, the expiration may be no
 // more than seven days in the future.
 // Required.
 Expires time.Time

 // ContentType is the content type header the client must provide
 // to use the generated signed URL.
 // Optional.
 ContentType string
```

Seemed a bit weird that the ContentType is marked as optional, but I assumed this meant that it didn't matter what content type was provided and that GCP would accept anything thrown at it. But as you will see later, this is not exactly the case, and the wording "optional" is a bit misleading.

Anyways, for now, everything looks good, at least for testing purposes. The next step was to update the GQL schema and connect the new function to a resolver, but I won't bore you with the details of all that. Let's jump straight to testing the new route with the most fundamental of tools: cURL!

```bash
$ curl -X PUT --upload-file FILE_TO_UPLOAD SIGNED_URL
```

As expected, everything looked fine, and the file was uploaded successfully over to the bucket. Alright then, let's tackle this on the client side now.

For the story's sake, I'll use vanilla JavaScript, even though the project wasn't originally built with it. (Nevertheless, I still had to write a vanilla JS version of the upload for debugging purposes.)

```javascript
document
  .getElementById("uploadForm")
  .addEventListener("submit", async function (event) {
    event.preventDefault();
    const file = this.file.files[0];
    try {
      const signedURLs = await getSignedURLs(1);
      const signed = signedURLs[0];
      const response = await fetch(signed, {
        method: "PUT",
        headers: {
          "Content-Type": file.type,
        },
        body: file,
      });
      if (!response.ok) {
        throw new Error("Network response was not ok");
      }
      const data = await response.json();
      console.log("Response data:", data);
    } catch (error) {
      console.error("There was a problem with the fetch", error);
    }
  });
```

For some reason, this simple fetch just didn't work, and the only error I got was 403 Forbidden. The first thing that came to mind was, "The signed URL must be broken," but this didn't make much sense, as I had just tested it with cURL, and everything was working fine. Then it must be a client-side issue, right?

After rummaging around StackOverflow and a few other tech forums and copy-pasting file upload code that was, in essence, the same as mine, I thought of inspecting their headers to see if they are, in fact, any different from what I was making in the code.

```bash
curl -v -X PUT --upload-file FILE_TO_UPLOAD SIGNED_URL
> User-Agent: curl/8.4.0
> Accept: */*
> Content-Length: 2340220
```

Nothing seems off here, just a simple HTTP request with minimal headers. Let's match it up with what the browser is doing.

```
Accept: */*
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: en-US,en;q=0.9,bg;q=0.8
Content-Length: 48355
Content-Type: image/jpeg
```

Hmm, the content type is the only difference, but it seems weird for it to be the problem as Google describes that option as "optional."

Well, anyway, let's try to remove it and see what happens.

```javascript
const response = await fetch(signed, {
  method: "PUT",
  body: file,
});
```

Still no luck. Let's check the headers again.

```
Accept: */*
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: en-US,en;q=0.9,bg;q=0.8
Content-Length: 48355
Content-Type: image/jpeg
```

This is peculiar. The browser automatically detects the content type and includes it in the request headers, even if I omit it in the fetch. What if we set the content type to explicitly be an empty string in the fetch?

```javascript
    headers:{
        "Content-Type": ""
    }
```

Hurray! Now the client-side request goes through, and the file is uploaded. This implies that the "optional" content type in the Google Storage SDK doesn't mean that all files are accepted, but rather that if omitted, only requests with Content-Type, either completely omitted or set to an empty string, go through. Quite misleading if you ask me.

Admittedly, sending files with an empty content type isn't particularly practical. As a result, I had to change my GenerateSignedUrl function by including the content type of the file as an input. But these are just a some particularities in this bizarre problem.
