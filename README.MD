# Mini Social

This is a mock social media platform created for the Social Computing course at the University of Oulu by Dr. Aku Visuri and Daniel Szabo.

The codebase contains three missing implementations, which you find [at end of app.py](https://github.com/Crowd-Computing-Oulu/mini_social_exercise/blob/44117e36e609b52c3452226797a0fd3fb5ff5258/app.py#L886). These are part of the coding assignments #1 and #2.

The repository contains a ready database, you do not have to create your own data.

## Participation features (Implementations A, B, C)

On top of the base platform, this fork implements three gamification and participation features (design rationale in `var/tier_system_plan.md`, review and audit trail in `var/tier_system_review.md`):

## Implementation A — Monthly participation tiers

Every user starts at the **Base** level (no badge shown). Within each calendar month:

- **Silver** level: at least **50 comments AND 100 reactions** → 🥈 badge next to the author's name on every comment.
- **Gold** level: at least **100 comments AND 200 reactions** → 🥇 badge (replaces Silver).
- Both conditions must be met (e.g. 100 comments with only 80 reactions is *not* enough for Silver).

Levels are computed dynamically (functions `get_user_badge()` and `get_badges_for_users()` in `app.py`), so a badge appears as soon as the threshold is reached and is lost again if comments/reactions are deleted.

## Implementation B — Monthly reset and activity summary

- Tiers use the current-month window, so counts are zeroed automatically when a new month begins (no reset job needed).
- On the first feed visit of the new month, a summary card shows how many comments and reactions the user made in the month just ended and which level they reached (`get_previous_month_summary()` in `app.py`, card in `templates/feed.html.j2`).

## Implementation C — "Question/Comment of the Day"

Once a day, the system automatically selects one user and asks them — through a private banner visible only to them — to start that day's discussion:

- The invite is **non-binding**: the user can decline without a reason, with no penalty, and with no effect on profile, level or future selection chances (users invited in the last 30 days are excluded from selection).
- If the user accepts, their next post created within the day is published as the **pinned post**: it stays at the top of the feed in every sort order and has a distinctive visual highlight ("📌 Discussione Ufficiale del Giorno").
- Key routes/functions: `get_daily_invite_status()`, `respond_to_daily_invite()`, `/daily_invite/respond` in `app.py`.

### Design choices worth knowing

- The "private notification" is implemented as a banner shown only to the selected user (the app has no notification system).
- The monthly summary is generated lazily on the first feed visit of the new month, not by a scheduled job.
- The selected user sees the invite until they answer; after a refusal no new user is invited that same day (one invite per day).
- All tier boundaries use UTC timestamps to match the ones written by SQLite.

### Testing

An automated smoke test runs the whole system (schema migration, thresholds, badges, invite flow, pinned post) against a temporary **copy** of the database:

```
python var/smoke_test_tier.py
```

Manual testing tips: thresholds refer to the current calendar month, so to see a badge quickly, add ~50 comments and ~100 reactions as a test user during the current month, or adjust `MONTHLY_TIERS` in `app.py` for local trials.

## Implementation D — Admin strikes & temporary suspension

An admin can send a **strike** on a post from the Admin dashboard (Posts tab,
"⚡ Invia Strike"): the post content is replaced with
`[content removed due to severe violation]` and a strike row is recorded against
the author. The author sees a one-time banner on their next feed visit. Three
strikes within a rolling 30-day window temporarily lock the login.
Key pieces: `strikes` table (migrated by `init_db_schema()`),
`POST /admin/posts/<post_id>/strike`, suspension check in `login()`,
`get_strike_notifications()`.

## Usage (for students)

1. Create a [fork](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo) of this repository
2. Clone your forked repository onto your computer
3. Make sure you have Python 3.11 installed
4. Install dependencies by running `pip install -r requirements.txt`
5. Start the server by running `pyhon ./app.py`
6. The web application will be available at http://127.0.0.1:8080. 
7. The admin interface is at http://127.0.0.1:8080/admin, available only to the Admin user (username: admin, pw: admin)
8. Make sure your fork is public, and add its URL to your coursework assignments.

### Docker

If you prefer to run Mini-Social as a docker container, you can build the image and run it as follows:

1. Build image with `docker build --tag mini-social .`
2. Run image with the following command, where 8000 is the port you will access the application at `http://127.0.0.1:8000`

```
docker run -d -p 8000:5000 \
    --restart always \
    -v $(pwd)/database.sqlite:/python-docker/database.sqlite \
    mini-social
```
 
