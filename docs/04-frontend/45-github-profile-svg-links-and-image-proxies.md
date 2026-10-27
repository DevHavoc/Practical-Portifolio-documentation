# Fixing Broken Icons in a GitHub Profile README

## Problem

Some technology icons in my GitHub profile were broken, while icons from another service still loaded. The README showed their alternative text instead of the images.

## What I found

The failing Iconify URLs used a colon between the icon collection and name. In the check, they returned HTTP 403 and HTML. Changing that separator to a slash returned HTTP 200 and an SVG image. This matches the [Iconify SVG endpoint format](https://iconify.design/docs/api/svg.html).

## What changed

I fixed 13 image URLs in the README. The icons, colors, sizes and surrounding profile text stayed the same. There were 12 different Iconify URLs because the GitHub icon appeared twice.

## What was checked

The README contained 27 images using 26 different URLs. All 26 URLs returned valid SVG images. After publishing, I also loaded the README rendered by GitHub and checked its 12 different Iconify proxy URLs. All returned successful image responses.

This checked both the image service and the path GitHub uses to display the images. The icons loaded at that time; future availability still depends on the image service.

## Code example

```html
<img src="https://api.iconify.design/bi/github.svg?color=%236f42c1" alt="github" width="36" height="36"/>
```
