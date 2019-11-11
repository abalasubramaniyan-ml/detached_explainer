# Page Decomposition

[//]: # (## Authors:)
[//]: # ([Author 1])
[//]: # ([Author 2])
[//]: # ([etc.])

[//]: # (Table of Contents [if the explainer is longer than one printed page])
[//]: # ([You can generate a Table of Contents for markdown documents using a tool like doctoc.])

## Introduction
[//]: # ([The “executive summary” or “abstract”. Explain in a few sentences what the goals of the project are, and a brief overview of how the solution works. This should be no more than 1-2 paragraphs.])

With the advent of virtual and augmented reality, a new style of 3D interfaces is emerging.
We are no longer limited to flat screens; instead the user interface can be broken up in sections and positioned within the user's 3D space (for VR) or the real world (for AR).

Browser vendors are developing WebXR for WebGL rendered content and are experimenting with [basic 3D models](https://developers.google.com/web/updates/2019/02/model-viewer) and other [interactive experiences](https://creator.magicleap.com/learn/guides/prismatic-getting-started).
However, until now, positioning CSS generated content in 3D was not possible.

## Proposal

Our goal is to break off parts of web page into the 3D space around the browser surface.

This would make viewing web pages a much more immersive experience rather than looking at a rectangular surface.
We hope to accomplish this by introducing a small addition to CSS.

We propose to introduce a new value for `transform-style`: `detached`.

This new property builds upon the existing 3D transforms that are shipping in all browsers and extends the behavior of `transform-style: preserve-3d`.
The main difference is that instead of flattening the transformed elements back to the page, the transformed element stays in 3D space.
If `transform-style: detached` is specified on an element, children with 3D transforms will be lifted out of the page.

Example:

    <!DOCTYPE html>
    <html>
    <head>
      <style>
        .scene {
            transform-style: detached;
        }
        .element {
            transform: rotateY(30deg) translateZ(25px); }
        }
      </style>
    </head>
    <body>
        <div id="scene">
            <div id="element">This is detached text</div>
        </div>
    </body>

Here is an example rendering of the effect:
![scene](https://github.com/rcabanier/detached_explainer/raw/master/detached.gif "Scene")

As the page or a parent element scrolls, the detached element should scroll.
CSS animations and transitions will apply as well.

In order to disallow unrestricted access of user's space to the content developer, we also propose to define a "safe space" that shall be requested to the user by content developer and user consented.
Content can not be placed outside this space.
![prism](https://github.com/rcabanier/detached_explainer/raw/master/prism.gif "Prism")

[//]: # (## Goals [or Motivating Use Cases, or Scenarios])
[//]: # ([What is the end-user need which this project aims to address?])

#
## Key scenarios
[//]: # ([If there are a suite of interacting APIs, show how they work together to solve the key scenarios described.])

### 1. Transform-style `detached` behaves like `preserve-3d` on platforms which do not support page decomposition.

We don't want authors to design different CSS for immersive vs flat browsers.
If a user agent renders on a flat surface, transform-style `detached` falls back to `preserve-3d` which will render the page visually the same.

[//]: # ([Description of the end-user scenario])

[//]: # (// Sample code demonstrating how to use these APIs to address that scenario.)

### 2. Nodes with transform-style: `flat` prevent descendents from being detached.

Transform style `flat` and effects that cause flattening ([spec](https://drafts.csswg.org/css-transforms-2/#grouping-property-values)) will cause `detached` fall back to `preserve-3d` behavior in descendent elements.

The one exception is root node which allows detached descendants even though an implicit `flat` transform-style is applied.

### 3. Only the top-most frame can use this feature.

iframes should not be allowed to created detached content.
This feature will only be available to the main frame.
In iframes, `detached` will fall back to `preserve-3d`.

### 4. Detached surfaces will not participate in stacking contexts.

Detached surfaces will be allowed to intersect with and pass through elements in the parent (and ancestor) stacking contexts.

Current browsers exhibit conflicting behavior in this regard - Rotated elements pass through sibling elements from the same stacking context in Safari, while in Chromium, they do not.

### 5. When detached elements are nested, only the top most element will signal that its children at detached

Allowing detached descendants of detached nodes significantly complicates the implementation and introduces multiple corner cases.
We propose that descendents with `detached` are treated as `preserve-3d`.

[//]: # (TODO: We need to explain this, but I cant think of the best explanation)

#
## Considered alternate proposals
[//]: # ([This should include as many alternatives as you can, from high level architectural decisions down to alternative naming choices.])

### 1. Applying the detached style directly to the surface to be detached.

Another possibility is to apply the `detached` value to elements under the `preserve-3d` style. For instance:

    <!DOCTYPE html>
    <html>
    <head>
      <style>
        .scene {
            transform-style: preserve-3d;
        }
        .element {
            transform-style: detached;
            transform: rotateY(30deg) translateZ(25px); }
        }
      </style>
    </head>
    <body>
        <div id="scene">
            <div id="element">This is detached text</div>
        </div>
    </body>

While this format allows explicitly identifying detached surfaces, it does not follow the behavior of `flat` and `preserve-3d` which modify descendant behavior.

### 2. Allow detached surfaces to break out of flattened parents.

Stacking contexts and grouping properties define how elements are laid out and rendered on 2d surfaces.
Detached content could be separated out of main surface before any of these rules and effects are applied.

This allows for detached surfaces in more scenarios, but can be confusing to define how the elements are transformed if transforms are compounded. ie. If the parent is rotated with respect to root, how should the detached descendent be positioned?

#
## Feedback / Opposition
[//]: # ([Implementors and other stakeholders may already have publicly stated positions on this work. If you can, list them here with links to evidence as appropriate.])

### 1. Feedback from [csswg](https://github.com/w3c/csswg-drafts/issues/4242)

Concern was raised that a similar behavior could be achieved without introducing a new transform style.

In the alternative proposed by David Baron (with added details from Chris Harrelson), allowing for `transform-style: preserve-3d` on document root would allow for transformed descendents to be implicitly detached.

Any node with transform-style: `flat` specified either explicitly or due to a grouping property would prevent its descendants being detached.

[//]: # ([Stakeholder B] : No signals)
[//]: # ([Implementor C] : Negative)
[//]: # ([If appropriate, explain the reasons given by other implementors for their concerns.])

## References & acknowledgements
[//]: # ([Your design will change and be informed by many people; acknowledge them in an ongoing way! It helps build community and, as we only get by through the contributions of many, is only fair.])
[//]: # ([Unless you have a specific reason not to, these should be in alphabetical order.])

[//]: # (Many thanks for valuable feedback and advice from:)

Josh Carpenter : Illustrations from [slide deck](https://docs.google.com/presentation/d/1ORdKs1wNe7QysRYSBtmW8LnMTFRu69gEwyOSrjIaZyA)
