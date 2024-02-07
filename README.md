# The Dictator App

I built this quick demo to test out the speech-to-text dictation. It's live here: https://dictator-by-aron.netlify.app/

It uses Web Speech API: SpeechRecognition (or webkitSpeechRecognition) for listening, and SpeechSynthesis for speaking.

It can:

- type based on speech
- change input language
- font size becomes smaller as the text becomes longer
- read out what just has been typed (if supported... sadly it's not supported for Danish)

## Requirements

- npm

## Setup

1. Open a terminal window in the project directory.
2. Install the project dependencies: `npm install`.
3. Start the development server by running: `npm run dev`.
