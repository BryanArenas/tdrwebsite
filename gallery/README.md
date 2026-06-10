# Adding photos to the website gallery

The gallery lives on the homepage ("A day in the life at TDR"). It stays completely
hidden — section and nav links — until the first photo is added, so it's always safe
to deploy.

## How to add a photo (2 steps)

1. **Drop the image file into this `gallery/` folder.**
   Name it lowercase with hyphens, no spaces: `reading-circle-k5.jpg`, not
   `IMG_4032 Final (2).JPG`.

2. **Open `index.html`, search for `ADD PHOTOS HERE`,** and add one line inside the
   `GALLERY` list for each photo:

   ```js
   var GALLERY = [
     { src: 'gallery/reading-circle-k5.jpg', alt: 'Kindergarten students seated in a circle reading with their teacher', caption: 'Morning reading circle — K–5' },
     { src: 'gallery/science-lab.jpg',       alt: 'Middle school students doing a science experiment', caption: 'Hands-on science, 6th grade' },
   ];
   ```

   - `src` — the file path (always starts with `gallery/`)
   - `alt` — one sentence describing the photo for blind visitors (screen readers read this)
   - `caption` — optional; shows under the photo in the full-screen viewer
   - The order of the lines = the order on the page. Commit and push; done.

## Optimal image size (for fast loading at high quality)

| Setting | Value |
|---|---|
| **Dimensions** | **1600 px on the long edge** (e.g. 1600×1200). Phone photos come out 4000+ px — always resize down. |
| **Format** | JPEG (`.jpg`) |
| **Quality** | 75–82% |
| **Target file size** | **150–400 KB per photo** — never upload multi-MB originals |
| **Orientation** | Any works. The grid crops tiles to 4:3 (landscape shows best there); the full-screen viewer always shows the complete, uncropped photo. |

Easiest tool: **[squoosh.app](https://squoosh.app)** — free, runs in the browser, no
install. Drag photo in → Resize to 1600 wide → MozJPEG ~78% → save. Most phones can
also export at "Large" size from the share sheet, which is close enough.

About 10–20 photos is the sweet spot — enough variety without endless scrolling.
Aim for range: classrooms, worship, recess/PE, electives, events, the building.

## Privacy checklist (school-specific — read before publishing)

- Only use photos of students whose families signed a **media release** (check the
  enrollment packet records).
- **No student names in captions or filenames.** "Morning reading circle" — not
  "Mrs. Lee's class with Sofia M."
- Avoid anything showing schedules, pickup procedures, or door codes in the background.

## Quick test after adding

Open the site, the Gallery section should appear between Curriculum and Admissions.
Click a photo: the viewer opens — arrow keys / swipe move between photos, Escape closes.
