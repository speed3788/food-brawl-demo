# Launching Food Brawl

Ordered roughly the way you'll actually do it. Nothing here needs a backend.

## 1. Accounts and money

| What | Cost | Notes |
|------|------|-------|
| Apple Developer Program | $99/year | Required to ship to the App Store. Enrol as an individual unless you have a registered company — company enrolment needs a D-U-N-S number and takes longer. |
| Google Play Console | $25 once | Optional if you only want iOS first. |
| Google Cloud (Places API) | pay per call | Has a monthly free allowance. **Set a billing cap before you ship.** |

Enrol with Apple early — approval can take a few days, and everything else waits on it.

## 2. Identity

Decide these once; changing them later is painful.

- **App name** as it appears on the store. "Food Brawl" may or may not be taken — check in
  App Store Connect before you get attached to it.
- **Bundle ID**, e.g. `com.yourname.foodbrawl`. Permanent. Lowercase, reverse-domain.
- **Subtitle**, 30 characters. "Winner picks dinner" fits.
- **Category.** Food & Drink primary; Entertainment secondary.
- **Age rating.** 4+ — no ads, no user content, no gambling mechanics. Answer the
  questionnaire honestly; the "contests" question is about real prizes, which this isn't.

## 3. The in-app purchase

- One **non-consumable**, not a subscription. Product ID something like
  `com.yourname.foodbrawl.pro`. Price tier $0.99.
- Display name "Food Brawl Pro", description: one payment, pick tonight's game yourself, no
  daily draw.
- You must implement **Restore Purchases** and put it somewhere findable (the potting shed).
  Apple rejects non-consumables without it.
- Apple takes 15% under the Small Business Program (opt in — you qualify), otherwise 30%.
- Test with a **sandbox tester account** in App Store Connect. Sandbox purchases don't charge
  real money, and you can't test IAP in the simulator — use a real device.
- Reviewers must be able to reach the paywall. Since nothing is gated behind login, that's
  automatic; just mention it in the review notes.

## 4. Google Places, done safely

- Enable **Places API** (Nearby Search, Place Details, Place Photos) in Google Cloud.
- **Restrict the key** to your iOS bundle ID and Android package plus signing certificate.
  An unrestricted key in a shipped app gets scraped and runs up your bill.
- **Set a daily quota cap and a billing budget alert.** Do this on day one, not after a
  surprise.
- Terms you have to honour, already built into the design: show "Powered by Google" wherever
  results appear, don't cache or store place names, ratings or addresses (the almanac keeps
  only the food and date), and don't use Places data to build your own restaurant database.
- Never ship the key in a public repo. Environment variable at build time, and if the repo is
  public, keep the key out of it entirely.

## 5. Privacy

This is the easy part, because the app collects nothing.

- **App Privacy in App Store Connect:** "Data Not Collected", assuming you add no analytics.
  If you later add crash reporting or analytics, you must update this.
- **Privacy manifest** (`PrivacyInfo.xcprivacy`): required. Declare the reason codes for any
  API you touch — local storage access is the likely one.
- **Privacy policy URL is mandatory** even collecting nothing. A single page saying the app
  stores your name, ingredient, filters, guest names and race history on your device only,
  sends nothing to any server, and that restaurant search queries go to Google under their
  policy. A GitHub Pages page is fine.
- **No App Tracking Transparency prompt needed** — you don't track across apps.
- Note in the policy that deep links hand off to Maps, DoorDash and Uber Eats, which have
  their own policies.

## 6. Store assets

- **App icon**, 1024x1024 PNG, no transparency, no rounded corners (Apple rounds it). Use the
  existing wordmark or a single ingredient — it has to read at 40px.
- **Screenshots.** Required: 6.9" iPhone (1320x2868 or 1290x2796). Up to 10, and the first
  three are what people actually see. Suggested set: tonight's race card, a game mid-race,
  the result card with restaurants, the paddock with the room backing runners, the almanac.
  You can screenshot straight from a device build.
- **Description.** Lead with the problem, not the mechanic: nobody can decide where to eat,
  so let a corgi race settle it. Mention it works offline, needs no account, and that Pro is
  one payment.
- **Keywords**, 100 characters total, comma separated, no spaces after commas, don't repeat
  words from your title. Something like:
  `dinner,decide,restaurant,food,wheel,picker,random,race,group,family,couples,takeout`
- **Promotional text**, 170 characters, changeable without a new build.
- **What's New** for each update.
- An **App Preview video** is optional and worth it later, not for launch.

## 7. Submitting

1. Build a release archive in Xcode, upload to App Store Connect.
2. Push it to **TestFlight** first. Install on your own phone. Play twenty races. Background
   the app mid-race and come back. Kill the app and reopen it. Buy Pro in sandbox, delete the
   app, reinstall, restore. Try to break the custom-food input.
3. Add 2-3 friends as TestFlight testers for a week. They will find things you can't.
4. **Review notes** — write these; they cut rejections. Say: no account needed, everything
   stored locally, one non-consumable IAP that unlocks manual game selection, restaurant
   results come from Google Places, deep links open third-party apps when installed and fall
   back to their websites.
5. Submit. First review is typically a day or two. Rejections are usually fixable metadata
   issues, not a verdict on the app.

Common rejection causes that apply to you: missing Restore Purchases, privacy policy URL
missing or dead, screenshots showing content the app doesn't have, IAP not testable, or a
deep link that opens a broken page when the target app isn't installed.

## 8. After launch

- Watch crash reports in Xcode Organizer for the first week.
- Watch the Google Cloud billing page for the first month.
- Ship the two unbuilt games (River Race, Monkey Tree) as free updates — good reason for
  people to come back, and the game framework already supports them.
- The almanac is the natural place for a future "most-eaten this month" feature, still with
  no server.

## What I'd do differently only if you get traction

Nothing about the current architecture needs to change for the first ten thousand users. No
backend, no accounts, one storage key. Resist adding a server until something actually
requires one — shared rooms across phones would be the first real reason.
