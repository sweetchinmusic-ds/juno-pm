# Lovable Prototype · Juno

## Prototype link

https://lovable.dev/projects/d89ac9db-9f54-484e-a9da-54818f16676c

## What it demonstrates

taking unstructured, raw user transcript data and seamlessly converting it into prioritized insights and a formatted Markdown PRD within a single, unified view. It proves that the three-column layout is an effective way to visualize the AI's transformation of data from raw input to final output.

## Debrief

- **What worked:** Specific initial prompt persona, task, constraints, format, and interaction were all defined, so the first build landed close to the target.
- **What broke / felt like a toy:** Fake processing — the "Process" button runs a fixed 1.6s timer and returns mocked insights. It doesn't actually parse meaning from the transcript, so the demo collapses the moment a user pastes real text.
- **What I'd change next pass:** The UI looks like a real tool, but the engine behind it is static demo data and that is something i would like to change in the next pass.
