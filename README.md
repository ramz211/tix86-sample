# tix86-sample


Sample verification output from [Tix86](https://tix86.com) — a free civic alert service that sends San Diego residents an SMS or email before their block's street sweeping window, so they can move their car and avoid the $83.50 ticket. 

## The Data Problem Tix86 Solves

San Diego's official street sweeping schedule database has known discrepancies with the signs physically posted on blocks. A bad alert is worse than no alert — a resident who acts on incorrect schedule data and still gets a ticket immediately loses trust in the service. Tix86 verifies every block's schedule directly against the posted street signs, which are the legal source of truth.

## How Verification Works

The pipeline walks each of San Diego's 3,365 swept block centerlines using the **Google Street View Map Tiles API**, capturing panoramic tile strips every 20 meters. Each strip is sent to **Vertex AI (Gemini 3.1 Flash-Lite)** for sign detection — if a street sweeping sign is visible, the model returns a bounding box. The sign crop is then upscaled and sent back to Gemini for OCR. The extracted text is fuzzy-matched against the expected schedule from the city's database.

## About This Sample

`sample_results.csv` contains 15 verified blocks from the Pacific Beach neighborhood — all confirmed `MATCH 100%`, with both curb sides independently detected and confirmed via Street View. The **Sign Text (OCR'd via Street View)** column shows the literal text Gemini read from the physical sign; the **City Schedule (Standardized)** column shows what the city database record says. A `MATCH 100%` result means the two are in agreement.

Notable rows:
- **SS-022951 (2000–2099 Reed Ave)** — both the south-side (1st Thursday) and north-side (1st Wednesday) signs detected and confirmed in a single pass
- **SS-013755 (1700–1799 Hornblend St)** — school zone block; sign text reads "STREET SWEEPING 1st THURSDAY OF THE MONTH 7AM – 10AM"
- **SS-010315 (1600–1701 Emerald St)** — school zone block confirmed via "SCHOOL STREET SWEEPING 1ST THURSDAY EVERY MONTH 7AM – 10AM"

## Column Reference

| Column | Source | Description |
|---|---|---|
| Block ID (SAPID) | City of San Diego | Unique centerline segment ID from the city's street asset database |
| Street Block | City of San Diego | Address range and street name |
| City Schedule (Standardized) | City of San Diego | Expected sweep schedule from the city's official database |
| Curb Sides Expected | City DB | Which curb sides (SS/NS or ES/WS) the city expects to have signs |
| Curb Sides Confirmed (Street View) | Street View scan | Which sides Gemini actually detected a sweeping sign on |
| Verification Result | Pipeline | MATCH / MISMATCH / PARTIAL / NOT_FOUND / TIMEOUT |
| Match Score (%) | Pipeline | Fuzzy match confidence between OCR'd text and city schedule |
| AI-Extracted Schedule (Gemini Vision) | Gemini 3.1 Flash-Lite | Schedule components Gemini parsed from the sign image |
| Latitude / Longitude | Street View | Scan position where sign was detected |
| Street View Pano Date | Street View Metadata API | Date of the panorama used for verification |
| Sign Text (OCR'd via Street View) | Gemini 3.1 Flash-Lite | Verbatim text read from the physical sign in Street View imagery |

## Links

- Service: [tix86.com](https://tix86.com)
- Coverage: San Diego, CA (3,365 swept blocks; California statewide expansion in progress)
