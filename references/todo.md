# TODO Checklist for Athlete Biomechanics Improvement Platform

This checklist covers the major tasks needed to build out the platform. You can use it as a guide for your development process, marking each item as you complete it.

---

## P1: Project Setup

- [ ] **Backend Initialization**  
  - [ ] Choice of the pose estimation model
  - [ ] Choice of the standard angles, heights, etc to recommend for the user

- [ ] **Frontend Initialization**  
  - [ ] Initialize Svelte project
  - [ ] Dashboard to present results to the user after AI processing

## P2: Video Upload

- [ ] **Backend Upload Endpoint**  
  - [ ] Create '/videos/upload' route  
  - [ ] Handle file storage (local/remote)  
  - [ ] Validate file type and size  
  - [ ] Write tests for success and error cases

- [ ] **Frontend Upload Component**  
  - [ ] Create file input with drag-and-drop  
  - [ ] Display upload progress/status  
  - [ ] Test for successful uploads, error handling

## P3: Final Integration & Deployment

- [ ] **End-to-End Flow**  
  - [ ] Stitch together sign-up, login, video upload, model inference, metric calculation, and dashboard  
  - [ ] Write comprehensive end-to-end tests

- [ ] **Deployment**  
  - [ ] Dockerize backend or set up serverless  
  - [ ] Host Svelte frontend (static hosting or similar)  
  - [ ] Verify environment variables/config  
  - [ ] Final check on production run

Use this checklist to track your progress and ensure that you cover every essential piece of the build. Check off each item as you complete it to maintain a clear overview of the project’s status.
