---
layout: post
title:  "Browser screenshot tool with MediaStream API"
date:   2022-01-26
categories: js mediastream mediadevices
---

## Problem

Recently I was faced with a challenge to build something that allowed some specific users to extract some information from a website and save it. Ideally they should take a screenshot and upload the image, but for security reasons the access to the computer was limited including the permission to download or upload files, so how could we overcame this?

## Solution

**Browser screenshot tool.**

The answer to the problem was to build a simple tool to take screenshots within the browser. But how? MediaStream API to the rescue!

What I did consisted in streaming one tab content to another tab video element and capture one frame of the stream, turning it into an image and send the image to the server.

Let us first take a look at what MediaStream API is and which parts we needed to build our solution.

## **MediaStream API**

**MediaStream API** provides the interfaces and methods for working with streams and their constituent tracks. It is related to [WebRTC](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API) which provides support for streaming audio and video data.

Of all the interfaces that make up the MediaStream API we used the MediaDevices and MediaStream interfaces.

### **MediaDevices**

From MDN Web Docs:

> The **`MediaDevices`** interface provides access to connected media input devices like cameras and microphones, as well as screen sharing. In essence, it lets you obtain access to any hardware source of media data.

The media input we are interested for the screenshot tool is the screen sharing one, so we can obtain the its media content and get an image from it. This interface has several methods that we can call to prompt the user to select a display, portion of a display (such as a window or tab) or to turn on the camera and/or microphone providing a MediaStream object containing the media data.

### **MediaStream**

From MDN Web Docs:

> The **`MediaStream`** interface represents a stream of media content. A stream consists of several **tracks**, such as video or audio tracks. Each track is specified as an instance of `[MediaStreamTrack](https://developer.mozilla.org/en-US/docs/Web/API/MediaStreamTrack)`.

The MediaStream will contain the content of the stream, we will use it to feed the video element and get a frame and convert it to the finally image we want.

## Tool Example

The simple sample tool I built has three main pieces, the react screenshot tool component and two main methods, one to setup the preview of the image and one to capture an image from the preview.

### ScreenshotTool

The main component ScreenshotTool is fairly small, it has two buttons, one to start the image preview and one to capture the image in the preview element. To show the streaming content we have a video html element and to show the image captured we have an img element.

```TypeScript
...

export const ScreenshotTool = () => {
  ...

  return (
    <Fragment>
      <div>
        <button disabled={canCapture} onClick={startImagePreview}>
          Preview
        </button>
        <button disabled={!canCapture} onClick={captureImageInPreview}>
          Capture
        </button>
      </div>
      <div>
        <div>Preview 🎥</div>
        <video className="Frame" id="video" ref={videoRef} autoPlay></video>
      </div>
      <div>
        <div>Image 🖼️</div>
        <img
          className="Frame"
          alt="Screen capture will be displayed here"
          src={image}
          onClick={() => openImage(image)}
        ></img>
      </div>
    </Fragment>
  );
};
...
```

### ImagePreview

ImagePreview is one of the two main functions. It is called when the user clicks in the Preview button.

```TypeScript
export const imagePreview = async ({
  videoRef,
}: ImagePreviewInput): Promise<MediaStream | undefined> => {
  try {
    const videoElem = videoRef.current;
    if (!videoElem) throw Error("Video HTML element not defined");

    videoElem.srcObject = await navigator.mediaDevices.getDisplayMedia();

    return videoElem.srcObject;
  } catch (error) {
    console.error("imagePreview error: " + error);
  }
};
```

It takes as argument a reference to the video element that it used to set its source object with the MediaStream object that we get from calling the getDisplayMedia() and after selecting the tab or window we want to stream. By doing this the video html element starts to show the content of the stream, which in our case will be the content of one of the browsers tab.

### ImageCapture

ImageCapture is the function responsible for take the screenshot we want. In short it takes a look at the stream content in the video html element, grabs a frame and converts it to a png image.

```TypeScript
export const imageCapture = async ({
  videoRef,
}: ImageCaptureInput): Promise<string | undefined> => {
  try {
    const videoElem = videoRef.current;
    if (!videoElem) throw Error("Video HTML element not defined");

    let mediaStream = videoElem.srcObject as MediaStream;
    if (!mediaStream) throw Error("Video MediaStream not defined");

    const track = mediaStream.getVideoTracks()[0];
    const image = generateImageWithCanvas(track, videoElem);
    // const image = await generateImageWithImageCapture(mediaStreamTrack);

    mediaStream.getTracks().forEach((track) => track.stop());

    return image;
  } catch (error) {
    console.error("imageCapture error: " + error);
  }
};
```

We use the getVideoTracks function to get the stream frame, and then we use the frame to generate and png image using a canvas element. Instead of using a canvas element we tried to use the ImageCapture class and takePhoto function, but it didn't work because we can't take photos of muted tracks. For more details take a look [here](https://github.com/w3c/mediacapture-screen-share/issues/141) and [here](https://github.com/chromium/chromium/blob/master/third_party/blink/renderer/modules/imagecapture/image_capture.cc).

```TypeScript
const generateImageWithCanvas = (
  track: MediaStreamTrack,
  videoElem: HTMLVideoElement
) => {
  const canvas = document.createElement("canvas");

  const { width, height } = track.getSettings();
  canvas.width = width || 100;
  canvas.height = height || 100;

  canvas.getContext("2d")?.drawImage(videoElem, 0, 0);
  const image = canvas.toDataURL("image/png");

  return image;
};
```

### Demo

![demo]({{ site.baseurl }}/media/screenshot-tool/demo-screenshot-tool.mp4)

## Conclusion

We can use this amazing interfaces to easily build solution to our problems that initially may sound a bit complicate but most of the complicated parts are very well implemented and document in the MediaStream API. This article show a very specif problem and solution but I think it shows what you can do with the MediaStream API, even if you don't have a similar problem.