# GitHub Profile SVG Links and Image Proxies

## Problem

Several technology icons on the public profile README appeared broken, while icons from another CDN loaded. Their alternative text was visible, which narrowed the issue to image retrieval rather than missing Markdown sections.

## Investigation

The broken requests used a colon between the Iconify collection and icon name. During the investigation, those requests returned HTTP 403 and HTML rather than an SVG. The same identifiers requested with a slash returned HTTP 200 and an SVG content type. The [documented Iconify SVG endpoint](https://iconify.design/docs/api/svg.html) uses a collection/name path.

The correction replaced 13 image-source URLs without changing icon identifiers, palettes, dimensions or surrounding profile text. There were 12 unique Iconify URLs because the GitHub icon appeared more than once. The change affected one README file and was published to the profile repository.

## Verification

All 26 unique image URLs in the 27 image occurrences returned valid SVG content. Verification then fetched the README as rendered by GitHub and requested the 12 unique Iconify images through GitHub's own proxy. All returned successful image responses. Checking only a source URL in a local browser would not have verified the published proxy path.

## Result and limits

The icons loaded through the published README path at verification time. The evidence identifies the observed colon-path rejection; it does not claim that every 403 from the service has that cause or guarantee future third-party availability. The change is a public, traceable example of following a symptom through source markup, origin HTTP behavior and the hosting proxy.

## Code example

```html
<img src="https://api.iconify.design/bi/github.svg?color=%236f42c1" alt="github" width="36" height="36"/>
```
