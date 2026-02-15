# Requirements Document: Active Recall Engine

## Introduction

The Active Recall Engine is a video learning platform designed to combat "tutorial hell" by transforming passive video consumption into an active learning experience. The system monitors information density in educational videos and automatically pauses playback at critical moments to verify user understanding through multimodal assessments. By leveraging AWS services including Amazon Bedrock, Amazon Titan Image Generator v2, Amazon Transcribe, and AWS Elemental MediaConvert, the platform creates adaptive, visually-enhanced learning interventions that reinforce comprehension of complex engineering concepts.

## Glossary

- **System**: The Active Recall Engine platform
- **User**: A learner consuming educational video content
- **Video**: Educational content uploaded to the platform
- **Intervention_Point**: A timestamp in a video where the system pauses for assessment
- **Cognitive_Load_Analyzer**: The Bedrock-powered component that identifies high-complexity segments
- **Visual_Generator**: The Titan Image Generator v2 component that creates assessment diagrams
- **Micro_Review_Clip**: A custom video segment created for remediation
- **High_Element_Interactivity_Segment**: A video segment where multiple new concepts are introduced simultaneously
- **Multimodal_Ingestion_Pipeline**: The component that processes uploaded videos into analyzable formats
- **Assessment**: A question or interactive challenge presented to verify understanding
- **Transcript**: Time-stamped text representation of video audio
- **Video_Frame**: A single image extracted from video at a specific timestamp
- **Embedding**: Vector representation of video content for semantic analysis

## Requirements

### Requirement 1: Video Upload and Storage

**User Story:** As a content creator, I want to upload educational videos to the platform, so that learners can access them for active recall learning.

#### Acceptance Criteria

1. WHEN a user uploads a video file, THE System SHALL store it in Amazon S3
2. WHEN a video is uploaded, THE System SHALL generate a unique identifier for the video
3. WHEN a video upload completes, THE System SHALL confirm successful storage to the user
4. THE System SHALL accept video files in MP4, MOV, and AVI formats
5. WHEN a video exceeds 5GB in size, THE System SHALL reject the upload and provide an error message

### Requirement 2: Multimodal Content Processing

**User Story:** As a learner, I want the system to understand both the audio and visual content of videos, so that assessments are contextually relevant.

#### Acceptance Criteria

1. WHEN a video is stored in S3, THE Multimodal_Ingestion_Pipeline SHALL invoke Amazon Transcribe to generate time-stamped transcripts
2. WHEN transcription completes, THE System SHALL extract video frames at 1-second intervals
3. WHEN video frames are extracted, THE System SHALL process them using Amazon Bedrock Multimodal Embeddings
4. WHEN processing embeddings, THE System SHALL correlate visual events with transcript timestamps
5. WHEN a slide change is detected in video frames, THE System SHALL mark the timestamp as a visual transition point
6. WHEN multimodal processing completes, THE System SHALL store all extracted data with references to the original video

### Requirement 3: Cognitive Load Analysis

**User Story:** As a learner, I want the system to identify complex segments in videos, so that I receive help when concepts are most challenging.

#### Acceptance Criteria

1. WHEN a transcript is available, THE Cognitive_Load_Analyzer SHALL analyze it using Amazon Bedrock
2. WHEN analyzing transcripts, THE Cognitive_Load_Analyzer SHALL apply Cognitive Load Theory principles to identify High_Element_Interactivity_Segments
3. WHEN multiple new concepts are introduced within a 60-second window, THE Cognitive_Load_Analyzer SHALL classify that segment as high cognitive load
4. WHEN a High_Element_Interactivity_Segment is identified, THE System SHALL mark its end timestamp as an Intervention_Point
5. WHEN cognitive load analysis completes, THE System SHALL store all Intervention_Points with their associated video
6. THE Cognitive_Load_Analyzer SHALL identify at least one Intervention_Point for every 5 minutes of video content

### Requirement 4: Adaptive Video Playback

**User Story:** As a learner, I want videos to pause automatically at key moments, so that I can verify my understanding before continuing.

#### Acceptance Criteria

1. WHEN video playback reaches an Intervention_Point timestamp, THE System SHALL pause the video
2. WHEN the video pauses, THE System SHALL display an assessment to the user
3. WHEN a user completes an assessment, THE System SHALL resume video playback
4. WHEN a user requests to skip an assessment, THE System SHALL record the skip event and resume playback
5. THE System SHALL maintain playback state across browser sessions for authenticated users

### Requirement 5: Multimodal Assessment Generation

**User Story:** As a learner, I want assessments that include visual diagrams, so that I can engage with spatial and visual concepts effectively.

#### Acceptance Criteria

1. WHEN an Intervention_Point is reached, THE System SHALL generate an assessment based on the preceding video segment
2. WHEN generating assessments, THE System SHALL use Amazon Bedrock to create contextually relevant questions
3. WHEN the video content involves spatial concepts, THE Visual_Generator SHALL use Amazon Titan Image Generator v2 to create visual diagrams
4. WHEN creating visual assessments, THE Visual_Generator SHALL use image conditioning techniques including Canny Edge or Segmentation
5. WHEN a visual diagram is generated, THE System SHALL create fill-in-the-blank or interactive elements requiring user input
6. WHEN generating questions for non-spatial concepts, THE System SHALL create text-based questions
7. THE System SHALL generate assessments within 3 seconds of reaching an Intervention_Point

### Requirement 6: Visual Assessment Examples

**User Story:** As a learner studying data structures, I want visual diagrams with missing elements, so that I can actively reconstruct concepts rather than passively recognize them.

#### Acceptance Criteria

1. WHEN the video discusses linked lists, THE Visual_Generator SHALL create node-and-arrow diagrams with missing connections
2. WHEN the video discusses tree structures, THE Visual_Generator SHALL create tree diagrams with missing nodes or edges
3. WHEN the video discusses memory layouts, THE Visual_Generator SHALL create memory diagrams with missing pointer arrows
4. WHEN the video discusses graph algorithms, THE Visual_Generator SHALL create graph visualizations with missing edges or labels
5. WHEN generating visual assessments, THE System SHALL ensure exactly one element is missing or requires completion

### Requirement 7: User Response Collection

**User Story:** As a learner, I want to provide answers through multiple input methods, so that I can respond naturally to different types of questions.

#### Acceptance Criteria

1. WHEN a text-based assessment is displayed, THE System SHALL provide a text input field for user responses
2. WHEN a visual assessment is displayed, THE System SHALL provide drawing or selection tools for user interaction
3. WHEN a user submits a response, THE System SHALL validate the input is not empty
4. WHEN a user submits a response, THE System SHALL store the response with timestamp and assessment identifier
5. THE System SHALL provide a submit button that is disabled until valid input is provided

### Requirement 8: Answer Evaluation

**User Story:** As a learner, I want immediate feedback on my responses, so that I know whether I understood the material correctly.

#### Acceptance Criteria

1. WHEN a user submits a response, THE System SHALL evaluate it using Amazon Bedrock
2. WHEN evaluating text responses, THE System SHALL use semantic similarity rather than exact string matching
3. WHEN evaluating visual responses, THE System SHALL compare user input against the expected diagram structure
4. WHEN a response is correct, THE System SHALL display positive feedback and resume video playback
5. WHEN a response is incorrect, THE System SHALL display the correct answer and trigger Micro_Review_Clip generation
6. THE System SHALL complete evaluation within 2 seconds of response submission

### Requirement 9: Micro-Review Clip Generation

**User Story:** As a learner who answered incorrectly, I want to review the exact moment where the answer was explained, so that I can reinforce my understanding efficiently.

#### Acceptance Criteria

1. WHEN a user provides an incorrect response, THE System SHALL identify the video segment containing the correct answer
2. WHEN the answer segment is identified, THE System SHALL use AWS Elemental MediaConvert to extract a 30-second clip
3. WHEN creating the Micro_Review_Clip, THE System SHALL slow down playback speed to 80% of original speed
4. WHEN the Micro_Review_Clip is ready, THE System SHALL present it to the user before resuming main video playback
5. WHEN a user completes watching a Micro_Review_Clip, THE System SHALL resume the main video from the Intervention_Point
6. THE System SHALL generate Micro_Review_Clips within 5 seconds of incorrect response evaluation

### Requirement 10: Learning Analytics

**User Story:** As a learner, I want to track my performance over time, so that I can identify areas needing improvement.

#### Acceptance Criteria

1. WHEN a user completes an assessment, THE System SHALL record whether the response was correct or incorrect
2. WHEN a user completes a video, THE System SHALL calculate an overall comprehension score based on assessment performance
3. WHEN a user views their profile, THE System SHALL display assessment accuracy rates by video
4. WHEN a user views their profile, THE System SHALL display topics where they frequently answer incorrectly
5. THE System SHALL store all assessment attempts with timestamps for historical analysis

### Requirement 11: Content Creator Dashboard

**User Story:** As a content creator, I want to see how learners perform on my videos, so that I can improve my teaching materials.

#### Acceptance Criteria

1. WHEN a content creator views their dashboard, THE System SHALL display aggregate assessment performance for each video
2. WHEN viewing video analytics, THE System SHALL show which Intervention_Points have the lowest success rates
3. WHEN viewing video analytics, THE System SHALL display average time spent on each assessment
4. WHEN multiple learners have completed a video, THE System SHALL calculate and display median comprehension scores
5. THE System SHALL update analytics data within 1 minute of new assessment completions

### Requirement 12: User Authentication and Authorization

**User Story:** As a user, I want secure access to my learning progress, so that my data remains private.

#### Acceptance Criteria

1. WHEN a user registers, THE System SHALL create an account with email and password
2. WHEN a user logs in, THE System SHALL verify credentials and create an authenticated session
3. WHEN a user is authenticated, THE System SHALL allow access to their learning history and progress
4. WHEN a user is not authenticated, THE System SHALL restrict access to video playback and assessments
5. WHEN a user logs out, THE System SHALL terminate the session and clear authentication tokens

### Requirement 13: Video Playback Controls

**User Story:** As a learner, I want standard video controls, so that I can navigate content at my own pace between intervention points.

#### Acceptance Criteria

1. THE System SHALL provide play, pause, and seek controls for video playback
2. WHEN a user seeks to a timestamp, THE System SHALL update playback position accordingly
3. WHEN a user adjusts volume, THE System SHALL persist the volume setting across videos
4. WHEN a user enables fullscreen mode, THE System SHALL maintain assessment functionality
5. WHEN a user seeks past an Intervention_Point, THE System SHALL mark that assessment as skipped

### Requirement 14: Error Handling and Resilience

**User Story:** As a learner, I want the system to handle failures gracefully, so that temporary issues don't disrupt my learning experience.

#### Acceptance Criteria

1. WHEN Amazon Transcribe fails to process a video, THE System SHALL retry up to 3 times before notifying the user
2. WHEN Amazon Bedrock is unavailable, THE System SHALL queue analysis requests and process them when service is restored
3. WHEN Titan Image Generator fails to create a visual, THE System SHALL fall back to text-based assessments
4. WHEN AWS Elemental MediaConvert fails, THE System SHALL display the correct answer without a Micro_Review_Clip
5. WHEN any AWS service returns an error, THE System SHALL log the error with context for debugging
6. WHEN network connectivity is lost during video playback, THE System SHALL cache the current position and resume when connectivity is restored

### Requirement 15: Performance and Scalability

**User Story:** As a platform administrator, I want the system to handle multiple concurrent users efficiently, so that the platform scales with demand.

#### Acceptance Criteria

1. WHEN 100 users are watching videos simultaneously, THE System SHALL maintain video playback latency below 2 seconds
2. WHEN processing video uploads, THE System SHALL handle at least 10 concurrent uploads without degradation
3. WHEN generating assessments, THE System SHALL complete generation within 3 seconds for 95% of requests
4. THE System SHALL use Amazon S3 CloudFront distribution for video delivery to minimize latency
5. THE System SHALL implement caching for frequently accessed video metadata and transcripts
