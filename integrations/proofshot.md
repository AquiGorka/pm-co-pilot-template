# ProofShot Integration (Video Proof)

## What it does

Records screen sessions as video proof of UI features working. Captures browser interaction and generates a video file that can be attached to PRs, grant submissions, or documentation.

## Install

```bash
npm install -g proofshot
```

## Usage

### 1. Start recording

```bash
proofshot start --output proof-[FEATURE].mp4
```

### 2. Drive the browser

Open the app, interact with the feature you want to prove. ProofShot records the screen.

### 3. Stop and generate

```bash
proofshot stop
```

Produces the video file at the specified output path.

### 4. (Optional) Post to a PR

Attach the video to a GitHub PR as a comment or upload to object storage and link it.

## When to Use

- Proving a UI feature works for stakeholders or grant reviewers
- Demonstrating a flow end-to-end before/after a change
- Recording regression tests visually
- Creating visual documentation for onboarding

## Artifacts

- `.mp4` video file of the recorded session
- Timestamped, reproducible proof of feature behavior

## Gotchas

- Requires a display/screen to record. Doesn't work in headless environments without a virtual display.
- Large recordings can produce big files. Keep sessions focused on the specific feature.
- Browser must be fully loaded before starting the recording for clean output.
