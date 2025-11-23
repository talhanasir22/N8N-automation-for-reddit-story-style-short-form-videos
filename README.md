# 🎥 Reddit → Short-Form Video Generation Workflow  
Automated TTS + Captions + Video Composition (n8n)

This n8n workflow automatically creates polished, short-form videos (TikTok, Reels, Shorts) from text inputs.  
It combines a user-uploaded background video with Text-to-Speech audio and auto-generated captions—then saves the final video URL to Baserow.

---

## 📌 Workflow Name
**Reddit - Video**

---

## 🚀 Purpose
Generate high-quality, vertically oriented **1080×1920** videos by combining:

- A user-uploaded background video  
- Text-to-Speech audio (from Title + Body script)  
- Automatically generated dynamic captions  
- (Optional) a title card  
- Automatic logging to Baserow

---

## 🛠️ Prerequisites

### Platforms / Services Required
- **n8n Instance** – to execute the workflow  
- **S3 / MinIO Storage** – stores:  
  - Uploaded background video  
  - Generated TTS audio  
  - Final rendered video  
- **Baserow Database** – tracks each video's processing status & URLs

### External Media Processing APIs
These endpoints require an `x-api-key` header:

| Purpose | Endpoint |
|--------|----------|
| Text-to-Speech | `http://173.249.21.108:8880/v1/audio/speech` |
| FFmpeg Video Composition | `http://173.249.21.108:8080/v1/ffmpeg/compose` |
| Caption Generation | `http://173.249.21.108:8080/v1/video/caption` |

---

## 🔄 High-Level Workflow Steps

1. **Input & Upload**  
   User submits Title, Body text, voice selection, and background video.

2. **Database Tracking**  
   Creates a new row in Baserow, sets status to **Incomplete**.

3. **TTS Audio Creation**  
   Title + Body → TTS API → audio file.

4. **Video Assembly**  
   Background video + audio are merged using FFmpeg  
   Output video is scaled to **1080×1920 (vertical)**.

5. **Captioning**  
   Dynamic subtitles are added using captioning API  
   (The Bold Font, bottom-center, profanity filtered).

6. **Finalization**  
   Final video uploaded to S3  
   Baserow record updated to **Completed** with final URL.

---

## ⚙️ Node Breakdown

| Node Name | Type | Description |
|-----------|------|-------------|
| **Form (Trigger)** | formTrigger | Collects Title, Body, Voice, and Background Video. |
| **Video Uploader** | s3 | Uploads background video to S3 as `bg_<submittedAt>.mp4`. |
| **Saving input to Baserow** | baserow | Creates Baserow record with title, body, video URL, and status `Incomplete`. |
| **Mix voices** | httpRequest | Sends Title + Body text to TTS API to generate voice audio. |
| **Upload TTS audio** | s3 | Uploads generated TTS audio to S3. |
| **Update Baserow** | baserow | Updates row with TTS audio URL. |
| **Getting rows** | baserow | Fetches video + audio URLs needed for video processing. |
| **Combine Clips** | httpRequest | Uses FFmpeg API to merge video + audio and format to 1080×1920. |
| **Add caption** | httpRequest | Creates dynamic captions using caption API (bottom-center, highlighted). |
| **Add final video to Baserow** | baserow | Updates record to `Completed` and adds final video URL. |

---

## ✅ Summary
This workflow automates the entire short-form content creation process—from raw text + video input to a fully edited, captioned final video, ready for TikTok/Reels/Shorts.

