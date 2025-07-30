# Project Specification: Athlete Biomechanics Improvement Platform

## Overview

The platform aims to help athletes improve their biomechanics, starting with cyclists. Users will upload a side-view video of themselves cycling, and an AI model will analyze their pose and provide recommendations for adjustments based on literature recommendations.

## Core Functionalities

1. **User Registration and Authentication**
   - Users can create an account and log in.
   - Secure password storage and authentication.

2. **Video Upload**
   - Users can upload a side-view video of themselves cycling.
   - Supported video formats: MP4, AVI, MOV.

3. **AI Analysis**
   - Use segmentation models for segmentation and pose estimation.
   - Analyze key metrics such as saddle height, handlebar position, knee angle, etc.

4. **Recommendations Dashboard**
   - Display analyzed metrics and recommended adjustments.
   - Visual representation of the cyclist's pose and suggested changes.

5. **User Interface**
   - A really nice and minimalist UI. The pose estimation allows to create a visual representation of the cyclist. The current and optimized positions are showed side by side. The user can drag different key points (such as the position of the saddle, etc...) in order to visualize the changes in real-time.

## Detailed Requirements

### User Registration and Authentication

- **Sign Up**
  - Fields: Username, Email, Password
  - Email verification
- **Log In**
  - Fields: Email, Password
  - Password reset functionality

### Video Upload

- **Upload Page**
  - Drag-and-drop or file selection for video upload
  - Progress bar for upload status
- **Backend Processing**
  - Store videos securely
  - Trigger AI analysis upon successful upload

### AI Analysis

- **Pose Estimation**
  - Use segmentation models to detect and segment the cyclist
  - Extract key points for pose estimation
- **Metric Calculation**
  - Calculate metrics such as:
    - Saddle height
    - Handlebar position
    - Knee angle
    - Hip angle
    - Ankle angle
- **Recommendation Generation**
  - Compare metrics against literature recommendations
  - Generate personalized adjustment suggestions

### Recommendations Dashboard

- **Metrics Display**
  - Show calculated metrics with visual aids
- **Adjustment Suggestions**
  - List recommended adjustments with explanations
- **Visual Representation**
  - Overlay suggested pose adjustments on the original video

## Overall style

- Minimalistic.
- Dark.

## Architecture Choices

- **Frontend**
  - Framework: Svelte
- **Backend**
  - Python.
  - AI Model: to be defined, see [here](./pose_estimation_model.md).
