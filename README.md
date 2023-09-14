# SafeUpload API Documentation

## Overview

The SafeUpload API allows developers to upload videos, check for NSFW content, and if the content is safe, upload the asset to Livepeer. This documentation provides a detailed overview of how to interact with the API using the provided script.

## Prerequisites

- Node.js
- npm packages: `axios`, `fs`, `form-data`

## Installation

1. Ensure you have Node.js installed.
2. Install the required npm packages:

```bash
npm install axios fs form-data
```

## API Endpoints

### 1. Upload Video

**Endpoint:** `https://safeupload.fly.dev/upload`

**Method:** POST

**Description:** Uploads a video file or a video URL.

**Payload:**

- If uploading a file: Use the `video` key with the file stream.
- If uploading a URL: Use the `videoUrl` key with the video URL.

**Response:** Returns an object with the `uploadPath` key, which contains the path to the uploaded video.

### 2. Check for NSFW Content

**Endpoint:** `https://safeupload.fly.dev/nsfwcheck`

**Method:** POST

**Description:** Checks the uploaded video for NSFW content.

**Payload:** 

- `videoPath`: The path to the uploaded video (obtained from the upload response).

**Response:** Returns an object with the `nsfwContent` key. If the length of `nsfwContent` is 1, no NSFW content was detected.

### 3. Upload Asset to Livepeer

**Endpoint:** `https://safeupload.fly.dev/uploadtolivepeer`

**Method:** POST

**Description:** If the video is safe, this endpoint uploads the asset to Livepeer.

**Payload:** 

- `name`: The name of the video (usually derived from the video path).
- `description`: A brief description of the video.
- `videoUrl`: The path to the uploaded video.

**Response:** Returns an object with the `asset` key, which contains details of the uploaded asset.

## Usage


```
const axios = require("axios");
const fs = require("fs");
const FormData = require("form-data");

const videoFilePath = "/Users/avinashnayak/Desktop/final/telugu.mp4";
// const videoFilePath =
//     "https://lp-playback.com/hls/a433owycjzdd0fkc/1920p0/fffebaab_1920p0.mp4";
const uploadURL = "https://safeupload.fly.dev/upload";
const generateSnapshotsURL = "https://safeupload.fly.dev/nsfwcheck";

const uploadAndGenerateSnapshots = async () => {
  try {
    const formData = new FormData();
    if (
      videoFilePath.startsWith("http://") ||
      videoFilePath.startsWith("https://")
    ) {
      // If it's a URL, just send the URL as a string
      console.log("Video is a URL. Sending URL to server...");
      formData.append("videoUrl", videoFilePath);
    } else {
      formData.append("video", fs.createReadStream(videoFilePath));
    }

    const uploadResponse = await axios.post(uploadURL, formData, {
      headers: formData.getHeaders(),
    });

    console.log("Upload successful!");
    console.log("Upload response data:", uploadResponse.data);

    const videoPath = uploadResponse.data.uploadPath;
    const generateSnapshotsResponse = await axios.post(generateSnapshotsURL, {
      videoPath,
    });

    console.log("Snapshots generation started successfully!");
    console.log(
      "Snapshots generation response data:",
      generateSnapshotsResponse.data
    );
    if (generateSnapshotsResponse.data?.nsfwContent?.length === 1) {
      console.log("NSFW content not detected. Uploading asset to Livepeer...");

      const assetResponse = await axios.post(
        "https://safeupload.fly.dev/uploadtolivepeer",
        {
          name: videoPath.slice(0, -4).split("/").pop(),
          description: "Test for NSFW content",
          videoUrl: videoPath,
        }
      );
      console.log("Asset uploaded to Livepeer");
      console.log(assetResponse.data.asset);
    } else {
      console.log("NSFW content detected. Asset not uploaded to Livepeer");
    }
  } catch (error) {
    console.error("Operation failed:", error.message);
  }
};
```

1. Determines if the video source is a file or a URL.
2. Uploads the video to the SafeUpload server.
3. Checks the uploaded video for NSFW content.
4. If no NSFW content is detected, uploads the asset to Livepeer.

Save the above code in test.js:

1. Set the `videoFilePath` variable to the path of your video file or the video URL.
2. Run the script:

```bash
node test.js
```

## Error Handling

The script contains basic error handling. If any operation fails, the error message will be printed to the console.

## Conclusion

This documentation provides a brief overview of the SafeUpload API and how to use the provided script to interact with it. Ensure you have the necessary prerequisites installed and follow the usage instructions to upload and check videos for NSFW content.
