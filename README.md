<h1 align="center">Link Preview JS</h1>

<div align="center">
<img src="https://user-images.githubusercontent.com/1634213/127743288-9f8a99f3-75d8-42c5-9a69-b47c661b7e18.jpg" height="500"/>

<pre align="center">
npm i link-preview-js
</pre>

<a align="center" href="https://github.com/ospfranco?tab=followers">
    <img src="https://img.shields.io/github/followers/ospfranco?label=Follow%20%40ospfranco&style=social" />
  </a>
  <br />
  <a align="center" href="https://twitter.com/ospfranco">
    <img src="https://img.shields.io/twitter/follow/ospfranco?label=Follow%20%40ospfranco&style=social" />
  </a>
</div>

Allows you to extract information from a HTTP url/link (or parse a HTML string) and retrieve meta information such as title, description, images, videos, etc.

## GOTCHAs

- You cannot request a different domain from your web app (Browsers block cross-origin-requests). If you don't know how _same-origin-policy_ works, [here is a good intro](https://dev.to/lydiahallie/cs-visualized-cors-5b8h), therefore **this library works on node (back-end environments) and certain mobile run-times (cordova or react-native)**.
- **This library acts as if the user would visit the page, sites might re-direct you to sign-up pages, consent screens, etc.** You can try to change the user-agent header (try with `google-bot` or with `Twitterbot`), but you need to work around these issues yourself.
- When you are testing this library do not use **google.com**, it does not return the necessary headers and you will think the library is broken.

## API

`getLinkPreview`: you have to pass a string, doesn't matter if it is just a URL or a piece of text that contains a URL, the library will take care of parsing it and returning the info of first valid HTTP(S) URL info it finds.

`getPreviewFromContent`: useful for passing a pre-fetched Response object from an existing async/etc. call. Refer to example below for required object values.

```typescript
import { getLinkPreview, getPreviewFromContent } from "link-preview-js";

// pass the link directly
getLinkPreview("https://www.youtube.com/watch?v=MejbOFk7H6c").then((data) =>
  console.debug(data)
);

////////////////////////// OR //////////////////////////

// pass a chunk of text
getLinkPreview(
  "This is a text supposed to be parsed and the first link displayed https://www.youtube.com/watch?v=MejbOFk7H6c"
).then((data) => console.debug(data));

////////////////////////// OR //////////////////////////

// pass a pre-fetched response object
// The passed response object should include, at minimum:
// {
//   data: '<!DOCTYPE...><html>...',     // response content
//   headers: {
//     ...
//     // should include content-type
//     content-type: "text/html; charset=ISO-8859-1",
//     ...
//   },
//   url: 'https://domain.com/'          // resolved url
// }
yourAjaxCall(url, (response) => {
  getPreviewFromContent(response).then((data) => console.debug(data));
});
```

## Options

Additionally you can pass an options object which should add more functionality to the parsing of the link

| Property Name                                                                          |                                             Result                                              |
| -------------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------: |
| imagesPropertyType (**optional**) (ex: 'og')                                           | Fetches images only with the specified property, `meta[property='${imagesPropertyType}:image']` |
| headers (**optional**) (ex: { 'user-agent': 'googlebot', 'Accept-Language': 'en-US' }) |                                Add request headers to fetch call                                |
| timeout (**optional**) (ex: 1000)                                                      |                                 Timeout for the request to fail                                 |
| followRedirects (**optional**) (default false)                                                      |                                 For security reasons, the library does not automatically follow redirects, a malicious agent can exploit redirects to steal data, turn this on at your own risk                                 |

```javascript
getLinkPreview("https://www.youtube.com/watch?v=MejbOFk7H6c", {
  imagesPropertyType: "og", // fetches only open-graph images
  headers: {
    "user-agent": "googlebot" // fetches with googlebot crawler user agent
    "Accept-Language": "fr-CA", // fetches site for French language
    // ...other optional HTTP request headers
  },
  timeout: 1000
}).then(data => console.debug(data));
```

## Response

Returns a Promise that resolves with an object describing the provided link.
The info object returned varies depending on the content type (MIME type) returned
in the HTTP response (see below for variations of response). Rejects with an error if response can not be parsed or if there was no URL in the text provided.

### Text/HTML URL

```javascript
{
  url: "https://www.youtube.com/watch?v=MejbOFk7H6c",
  title: "OK Go - Needing/Getting - Official Video - YouTube",
  siteName: "YouTube",
  description: "Buy the video on iTunes: https://itunes.apple.com/us/album/needing-getting-bundle-ep/id508124847 See more about the guitars at: http://www.gretschguitars.com...",
  images: ["https://i.ytimg.com/vi/MejbOFk7H6c/maxresdefault.jpg"],
  mediaType: "video.other",
  contentType: "text/html; charset=utf-8",
  videos: [],
  favicons:["https://www.youtube.com/yts/img/favicon_32-vflOogEID.png","https://www.youtube.com/yts/img/favicon_48-vflVjB_Qk.png","https://www.youtube.com/yts/img/favicon_96-vflW9Ec0w.png","https://www.youtube.com/yts/img/favicon_144-vfliLAfaB.png","https://s.ytimg.com/yts/img/favicon-vfl8qSV2F.ico"]
}
```

### Image URL

```javascript
{
  url: "https://media.npr.org/assets/img/2018/04/27/gettyimages-656523922nunes-4bb9a194ab2986834622983bb2f8fe57728a9e5f-s1100-c15.jpg",
  mediaType: "image",
  contentType: "image/jpeg",
  favicons: [ "https://media.npr.org/favicon.ico" ]
}
```

### Audio URL

```javascript
{
  url: "https://ondemand.npr.org/anon.npr-mp3/npr/atc/2007/12/20071231_atc_13.mp3",
  mediaType: "audio",
  contentType: "audio/mpeg",
  favicons: [ "https://ondemand.npr.org/favicon.ico" ]
}
```

### Video URL

```javascript
{
  url: "https://www.w3schools.com/html/mov_bbb.mp4",
  mediaType: "video",
  contentType: "video/mp4",
  favicons: [ "https://www.w3schools.com/favicon.ico" ]
}
```

### Application URL

```javascript
{
  url: "https://assets.curtmfg.com/masterlibrary/56282/installsheet/CME_56282_INS.pdf",
  mediaType: "application",
  contentType: "application/pdf",
  favicons: [ "https://assets.curtmfg.com/favicon.ico" ]
}
```

## License

MIT license
