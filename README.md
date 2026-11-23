# CCTV card for Home Assistant + Frigate

One card for a CCTV dashboard tab, laid out for both desktop and phone. It shows a live grid of your
cameras with **one timeline for all of them**. Drag the timeline back and **every camera replays the
same moment** from its Frigate recording.

| Desktop | Phone |
|---|---|
| <img src="images/desktop.png" width="560" alt="Desktop: big doorbell tile, four smaller tiles, an NVR panel, the timeline and an events strip"> | <img src="images/mobile.png" width="260" alt="Phone: doorbell full width, four small tiles two per row, the timeline and an events strip"> |

*(Mock-up: the real card with stock photos in place of the cameras, and made-up events. Photo credits at the bottom.)*

> **Note:** I built this card around my own setup: four cameras and a doorbell. For other numbers of
> cameras, see [More cameras](#more-cameras) for the `columns` settings that fill the grid with no gaps.

## What it does

- **Live grid.** One big camera with the rest small. Tap a small camera to make it the big one.
- **One timeline.** It covers 24 hours with the blue marker fixed in the middle.
  - grey = motion
  - amber = Frigate alert
  - purple = Frigate detection
  - the shaded right half is the future
- **Replay.** Drag the bar (or use the mouse wheel on desktop) back up to 10 days. Every tile plays its
  recording from the moment under the marker, and playback carries on until you press **Back to live**.
  Tapping the bar jumps straight to that time.
- **◀ ▶** (optional) step through **alerts**, newest first. ▶ past the newest alert returns to live.
- **Events strip** under the timeline: Frigate review thumbnails. Tap one to replay all cameras at that moment.
- **Status dot** on each camera label, same colours as the timeline: green when quiet, grey for motion,
  purple for a detection, amber for an alert. It reads Frigate's `binary_sensor.<camera>_motion` and
  `sensor.<camera>_review_status`, so wind or a parked car no longer keeps it amber.
- **NVR panel** (optional, desktop): Coral inference speed, disk, CPU, memory, uptime.
- **Sound button** (optional, per camera), e.g. for a doorbell. Cameras start muted and the button
  turns sound on for that tile. Leaving the page mutes it again.
- **Frigate button** (optional) bottom right, opens your Frigate UI in a new tab.
- **Light and dark**: follows each user's Home Assistant theme.

## What you need

- **[Frigate](https://frigate.video)** with recording on (continuous recording if you want to replay
  any moment, not just events).
- **[Frigate integration](https://github.com/blakeblackshear/frigate-hass-integration)** for Home Assistant.
  The card reads reviews, recordings and thumbnails through it.
- **[Advanced Camera Card](https://github.com/dermotduffy/advanced-camera-card)** 8.x. Each live tile is
  one of its cards.
- A browser that can play your cameras' codec. **H.265 recordings only replay in browsers that decode
  HEVC.**

Tested on Home Assistant 2026.11, Frigate integration 5.15.6, Advanced Camera Card 8.2.0, with five
cameras.

## Install

1. Copy [`cctv-card.js`](cctv-card.js) to `/config/www/cctv-card.js`.
2. **Settings → Dashboards → ⋮ → Resources → Add resource**: URL `/local/cctv-card.js?v=1`, type
   *JavaScript module*. Change the `?v=` number every time you update the file, or browsers keep the old one.
3. Add the cards. [`dashboard-example.yaml`](dashboard-example.yaml) is a whole view with both cards in it.
   [`desktop-example.yaml`](desktop-example.yaml) and [`mobile-example.yaml`](mobile-example.yaml) are the
   two cards on their own.

## Desktop and phone

Use two copies of the card in the same view, each with a screen-width `visibility` condition:

- **Desktop** (`min-width: 768px`): the default grid, 3 columns. The big tile takes 2×2 and the NVR panel
  fills the last cell.
- **Phone** (`max-width: 767px`): `layout: strip` with `columns: 2`. The big tile goes full width and the
  rest sit underneath, two per row, without labels so the picture isn't covered.
  `quality_by_size: true` keeps the small tiles on a lighter stream.

## More cameras

The big tile always takes a 2×2 block and the small tiles fill in around it, so some camera counts
leave an empty cell at the bottom. These settings fill the grid with no gaps:

| Cameras | Desktop (grid) | Phone (`layout: strip`) |
|---|---|---|
| 5 | `columns: 3` with `nvr`, or `columns: 4` without | `columns: 2` without `nvr` |
| 6 | `columns: 3` without `nvr`, or `columns: 5` with | `columns: 2` or `3` with `nvr` |
| 7 | `columns: 5` without `nvr` | `columns: 2` or `3` without `nvr` |
| 8 | `columns: 3` or `4` with `nvr` | `columns: 2` with `nvr` |
| 9 | `columns: 3` or `4` without `nvr` | `columns: 2` without `nvr` |

- More columns means smaller tiles. With 3 columns the big tile is two-thirds of the width, with 4 it's half.
- On a phone, `columns: 3` works but the small tiles get very small. `columns: 2` is easier to see.
- A gap isn't a problem, it's just one empty cell at the bottom of the grid.

## Options

| Option | Default | What it does |
|---|---|---|
| `cameras` | **required** | List of cameras (below). The first one starts big. |
| `columns` | `3` | Grid columns. |
| `layout` | grid | `strip` = big tile full width, small ones underneath (phone). |
| `nav_buttons` | `false` | Show ◀ ▶ to step through alerts. |
| `events` | `true` | `false` hides the events strip. |
| `event_style` | compact | `detail` = wider event cards with date, duration, camera and zones. |
| `nvr` | none | NVR panel entities (below). Leave out to hide the panel. |
| `quality_by_size` | `false` | Big tile plays the main stream, small tiles each camera's `sub` stream (via go2rtc). |
| `big_first` | `false` | When replaying, load the big tile first and the small ones after it. Feels quicker on a slow connection. |
| `frigate_url` | none | Shows a Frigate button that opens this URL. |
| `instance_id` | `frigate` | Frigate integration instance, if you have more than one. |
| `publish_height` | none | Shares the card's height as a CSS variable on the page (e.g. `cctv-live-height` → `--cctv-live-height`), so another card can match it. |

**Each camera:**

| Key | What it is |
|---|---|
| `entity` | The camera entity, e.g. `camera.driveway`. |
| `frigate` | The camera name in Frigate, e.g. `Driveway`. Also the go2rtc stream name for `quality_by_size`. |
| `name` | Label on the tile. |
| `motion` | Optional. Motion sensor for the label dot. Default: `binary_sensor.<frigate>_motion` (the `frigate` name in lower case). |
| `review_status` | Optional. Review status sensor for the label dot. Default: `sensor.<frigate>_review_status`. |
| `occupancy` | Optional, older setting. Only used when neither of the two sensors above exists: the dot turns amber while it is on. |
| `sub` | Optional. go2rtc stream name for the small tile with `quality_by_size`, e.g. `Driveway_sub`. |
| `audio` | Optional. `true` adds the sound button to this camera. |

**`nvr` panel:** `coral` (ms), `disk` (%), `cpu` (%), `mem` (GB), `mem_pct` (%), `uptime` (seconds, e.g.
`sensor.frigate_uptime`). Use whichever sensors you have for the machine Frigate runs on.

## Notes

- **Times are shown in `Europe/Dublin`.** It's set at the top of `cctv-card.js` (`const TZ`). Change it
  to your own time zone.
- Replay loads clips through the Frigate integration's recording proxy, 30 seconds at a time, and
  chains the next one when a clip ends.
- Timeline motion is loaded in 6-hour blocks. Past blocks are cached, and the current block refreshes
  every 5 minutes.
- Every dashboard save or resource change reloads open dashboards. That's Home Assistant, not the card.

## Photo credits

The screenshots use public-domain (CC0) photos, cropped and given a camera clock:
[front door](https://www.rawpixel.com/image/6038157/photo-image-public-domain-trees-wood),
[driveway](https://www.rawpixel.com/image/3290508/free-photo-image-outdoor-house-background-automobile-building) and
[street](https://www.rawpixel.com/image/3301349/free-photo-image-car-city-road-asphalt) from rawpixel;
[back deck](https://www.flickr.com/photos/71401718@N00/11283075513) and
[garden](https://www.flickr.com/photos/71401718@N00/9181680406) by Wonderlane on Flickr.

## License

MIT. Worked through with Claude.
