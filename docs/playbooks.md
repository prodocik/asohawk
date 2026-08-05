# Playbooks

Short recipes for common agent flows, built entirely from the tools in the [tool reference](../TOOLS.md). Paste one into your agent as-is, or just describe the goal; the tools' own contracts point the way.

## Onboard a new app

**When:** adding a new app and getting oriented, or a first run in a new workspace.

1. `search_appstore` or `add_app` puts the app in the workspace. Both are idempotent, safe to call again. Not live on the App Store yet? `import_asc_app` instead, from a connected App Store Connect account (see [Manage a pre-launch app](#manage-a-pre-launch-app)).
2. `set_countries` sets the storefronts to track.
3. `estimate_app_performance` on the app and 3 to 5 obvious competitors sizes the niche before spending calls on tracking.
4. `discover_competitors` or `run_discovery`, then `add_competitor` on the relevant results.
5. `discover_keyword_opportunities` and `inspect_metadata` build a starting keyword set, then `track_keywords` within quota.
6. `refresh_now` takes the first snapshot; `get_aso_health` and `get_recommendations` give a baseline report of where the app stands.
7. `record_learning` saves the starting picture; `create_task` for anything that needs a human, such as connecting App Store Connect or GA4, or shooting screenshots.

## Manage a pre-launch app

**When:** onboarding your own app before it has a public App Store listing, via a connected App Store Connect account.

1. `import_asc_app` with the ASC app id or bundle id imports it as pre-launch; `add_app`'s Lookup would fail here since there is nothing to look up yet.
2. `get_release_readiness`'s `data.prelaunch` checklist tracks what is still missing: ASO text and screenshots per locale, category, age rating, price, availability and the App Privacy questionnaire (non-authoritative, a human confirms it).
3. Fill in the listing the same way as for a live app: `propose_metadata_change` and/or `propose_screenshot_change`, then `apply_change` once a human approves.
4. `track_keywords` and `run_discovery` stay off limits meanwhile: no App Store listing means nothing to rank or discover keywords against yet.
5. Publication is detected automatically (a daily check), not a manual step. Once the app ships, `data.prelaunch` turns null and keyword tracking, rank collection and discovery all unlock on their own.

## Diagnose conversion

**When:** lots of impressions but few installs. Needs App Store Connect.

1. `inspect_acquisition` reads the impressions to product page views to downloads funnel.
2. Find the leaking step. Weak impressions to views points at icon, title, subtitle or first screenshot; weak views to install points at screenshots, description, rating or price.
3. `get_charts` and `inspect_competitor` show how competitors look at the same step.
4. `create_hypothesis` records the target metric and the expected direction.
5. Fix the leaking step with `propose_metadata_change` and/or `propose_screenshot_change`. Once a human approves, `apply_change` publishes it.

## Diagnose retention

**When:** users install but churn. Needs Google Analytics 4.

1. `inspect_retention` gives DAU/WAU/MAU, D1/D7/D30 cohorts, the first_open to onboarding to paywall to purchase funnel, and feature adoption.
2. Locate the drop-off step in that funnel.
3. If it looks like an expectations mismatch (good acquisition, poor D1), treat it as a listing problem: revisit it with the [Diagnose conversion](#diagnose-conversion) playbook or the screenshots directly. If it is a product problem, `create_task` for the developer and note the themes for the next whatsNew.
4. `record_learning` saves the diagnosis.

## Plan localization

**When:** deciding which languages to localize first.

1. `inspect_metadata` shows which locales the listing already fills.
2. `inspect_acquisition` shows top countries with traffic but no localized listing.
3. `inspect_competitor` shows which locales competitors ship.
4. Rank the gaps by traffic, missing locale and competitor activity.
5. `propose_metadata_change` with the new locales. The agent translates the copy itself; App Store Connect only validates length and locale codes.

## Harvest keywords

**When:** growing the tracked keyword set.

1. `discover_keyword_opportunities` surfaces candidates not yet tracked.
2. `inspect_keyword` filters candidates by difficulty and popularity.
3. `get_recommendations`: the New bucket is what to add.
4. `inspect_competitor`: keywords they rank for that this app does not.
5. `track_keywords` the best candidates; `archive_keywords` the dead ones to free quota.

## Spy a competitor's keywords

**When:** sizing up a competitor's keyword footprint, or looking for keywords they rank for that this app does not yet track. Paid plans only.

1. `discover_competitor_keywords` on the competitor: every keyword they rank for in a country, sourced from data ASOHawk already collects, with `i_track` flagging what this workspace's own apps already track.
2. Filter to `only_gap: true` for just the untracked keywords, already ordered by the competitor's best rank.
3. `inspect_keyword` on the strongest gap candidates to weigh difficulty and popularity before committing quota.
4. `track_keywords` the best candidates.

## Resolve keyword cannibalization

**When:** running 2 or more own apps that might be competing for the same keywords.

1. `inspect_keyword_cannibalization` finds terms where 2+ own apps currently rank in the same storefront.
2. Read each overlap's severity hint: contested means neither app has cleared the field, covered means one app already owns the term and the rest are trailing. It is a hint, not a verdict; decide case by case.
3. For a contested overlap worth fixing, reposition the trailing app onto a different term with `propose_metadata_change`. A human approves before it publishes.
4. `record_learning` saves which apps were repositioned and why.

## Daily briefing

**When:** starting a session and checking what changed since the last visit.

1. `get_events_since` with the stored cursor: domain events since the last visit (competitor metadata, version and price changes, own keywords entering or leaving the top 10, closed hypothesis windows).
2. `list_pending_changes`: what waits for a human approval, the most actionable item of the briefing.
3. `get_active_hypotheses`: which observation windows are close to their end.
4. `compare_periods` (week_over_week preset or a short window): how ranks, visibility and downloads moved. Its `known_changes` block lists releases and listing changes the platform recorded inside the window, yours and competitors', so moves come pre-attributed.
5. `list_tasks` (open, assigned to the user): what is stuck on a human.
6. Compose a short briefing: events first, then what needs a decision, then what moves on its own. For a notable event, name the matching playbook, such as investigate rank drop or competitor move.

## Investigate a chart move

**When:** a tracked app, yours or a competitor's, enters, exits or jumps in a top chart.

1. `get_charts` for the relevant country, chart type and genre: current standing for the app and its competitors.
2. `get_events_since`: chart_move events show who entered, who exited and who jumped by at least 10 spots between two collected snapshots.
3. A newcomer's rise: `estimate_app_performance` and `inspect_competitor` on it show what it is doing right.
4. Your own app's drop: `compare_periods` over the drop window (ranks, visibility, downloads) diagnoses the cause, the same way as the rank-drop playbook, on the chart axis. Check its `known_changes` first: a release or a competitor's listing change recorded inside the window often explains the move outright.
5. For a change worth trying, `create_hypothesis` then `propose_metadata_change`; a human approves it before it publishes. If the move looks like market noise instead, `record_learning` saves that call.

## Run a native A/B test

**When:** testing icon or screenshot variants with Apple's own Product Page Optimization instead of guessing. Needs iOS 15+ and a live app.

1. Prepare the variants: screenshots go through `request_screenshot_upload`; icon variants need alternate icons already compiled into the binary, so ask the human if they are not there yet.
2. `propose_ppo_test` with a name, the traffic split and up to 3 treatments.
3. A human always approves. This is high risk and never runs on auto, whatever the workspace's policy.
4. `apply_change` creates the experiment and starts it; the treatments then go through Apple's App Review.
5. `list_ppo_experiments` to watch its state and schedule.

## Change a product's price

**When:** repricing one in-app purchase or one subscription. Needs App Store Connect.

1. `inspect_products` lists the app's in-app purchases and subscriptions with their current prices and the exact price Apple offers today; take the product id from there.
2. `propose_metadata_change` with `iap_price` or `subscription_price`. One proposal carries one product price: the two fields cannot be combined, and repricing several products means several proposals.
3. `customer_price` must match an existing App Store Connect price point exactly, otherwise the proposal is refused with `INVALID_INPUT`. For an in-app purchase, `base_territory` must be the territory the product's price schedule is actually based on; changing which territory it is based on is not supported.
4. A human always approves. Price is revenue-affecting, so it is always high risk and never runs on auto, whatever the workspace's policy.
5. `apply_change` writes it to App Store Connect, then the platform re-reads the product and reports the published price back. For subscriptions, existing subscribers always keep their current price; that guarantee is Apple's and is not configurable here.
6. Applying a price equal to the one already live is refused with `PRECONDITION_FAILED` (Apple's 409 STATE_ERROR): nothing is written, and the price is most likely already live, possibly changed by hand while the change waited. Re-read it with `inspect_products` before proposing again.

## Publish a new build

**When:** the user hands you a built .ipa and wants it submitted for App Review. Needs App Store Connect.

1. Upload the .ipa yourself, under the user's own App Store Connect credentials: fastlane, Transporter or Xcode. Apple has no REST endpoint for binary upload, so this platform cannot do that step. On Linux, fastlane works through Transporter for Linux (set `FASTLANE_ITUNES_TRANSPORTER_USE_SHELL_SCRIPT=true`). Set `ITSAppUsesNonExemptEncryption` in Info.plist at build time, or the build gets stuck in Missing Compliance and cannot be submitted. `list_builds` shows the build numbers already taken, so pick a higher one.
2. `list_builds` until the uploaded build shows processing_state VALID. Apple's processing takes 5 to 30 minutes; the tool returns `retry_after_seconds` while the newest build is still processing. FAILED or INVALID means Apple rejected the binary: fix it and upload a new build, a failed one can never be attached.
3. `attach_build` with `build_id` (or `build_version`) attaches the VALID build to the editable version. If there is no editable version yet, pass the `version_string` the user asked for, verbatim; the platform never invents one on its own.
4. `get_release_readiness`: its `data.build` section and blockers list say what is still missing. Close each blocker with the matching tool (`propose_metadata_change` for texts, `propose_screenshot_change` for screenshots), each through the usual human approval.
5. `propose_release_submission` once readiness reports no blockers. Submitting to App Review is always high risk and always waits for a human in Approvals, whatever the workspace's agent permissions say.
6. After the human approves, `apply_change` sends the version to Apple's review queue; `get_change_status` reports the verification result a few minutes later. From submission on, only Apple decides the outcome.

## Analyze a native A/B test

**When:** checking on a running Product Page Optimization test, or wrapping up a finished one.

1. `list_ppo_experiments`: state, days elapsed and remaining, and any promoted winner.
2. `compare_periods` over the test window; this platform's data is aggregate, the per-treatment split only lives in App Store Connect's App Analytics.
3. Ask the human for the App Analytics conversion numbers; the API does not expose them.
4. A screenshots winner can be applied with `propose_screenshot_change`; an icon winner needs the icon made default in a new binary, so `create_task` for that.
5. `record_learning` and `close_hypothesis` save the outcome.
