# Moltour notes

## Website links

* [Moltour Heroku](http://moltour.herokuapp.com/biochem)
* [Old website](http://vr.vu-compmedchem.nl/)

## Options

separate app instead of mobile browser:

* viewer capabilities (zoom etc.) will be easier to conrol
* less hampered by browser compatibility and browser development issues/updates

WebVR already being replaced by WebXR. so if polyfill, should use [webxr-polyfill](https://github.com/immersive-web/webxr-polyfill)


NGL issues/possibilities for VR:

* stereocamera implemented in NGL, seems to be working. example on how ot use a different camera system. comment in question is [here](https://github.com/arose/ngl/commit/bfc251d302c3e88cefc23060865d72a93211e9e7)
    * however, does not seem to work properly on mobile due to some specific (OpenGL?) settings. jittering around objects with white background on iPhone.
* can use specific framework for VR that does device and camera for us (alternative to polyfill)
    * A-Frame -- easy to use, easy to set up VR environments, allows even merging of e.g. proteins and other 3D objects
        * [A-frame link](https://aframe.io/), [github branch](https://github.com/KJStrand/ngl/tree/AFRAME-NGL)
        * Mobile device examples: [https://amino-aframe.glitch.me](https://amino-aframe.glitch.me), [https://a-ngl.glitch.me](https://a-ngl.glitch.me)
        * however, same issues as stereocamera when using NGL on mobile (so probably internal NGL thing)

switch to [3Dmol.js](http://3dmol.csb.pitt.edu/index.html)

* graphically less advanced than NGL, but therefore provides better support for mobile, easier to extend (Albert)
* stereographical test with 3Dmol.js viewer worked perfectly
* could also potentially be combined with A-Frame

other links:

* [Chrome WebXR](https://developers.google.com/web/updates/2018/06/ar-for-the-web)
* [Microsoft WebXR](https://docs.microsoft.com/en-us/microsoft-edge/webvr/what-is-webvr)
* [Mozilla WebXR](https://hacks.mozilla.org/2018/09/webxr/)
* [Mozilla WebXR for iOS](https://itunes.apple.com/us/app/webxr-viewer/id1295998056)
