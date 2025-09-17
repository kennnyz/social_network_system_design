## Social Network System Design

## Functional requirements
- Create, read, update, delete posts
- Create, read, update, delete reactions/likes on posts
- Create, read, update, delete comments on posts
- Follow and unfollow users
- Generate a home feed based on follow graph in reverse chronological order
- Retrieve a user's feed and browse posts
- Search for popular places and view related posts

## Non‑functional requirements
- Audience and scale
  - 10,000,000 daily active users (DAU), CIS region only
  - Average per user: 20 read requests/day; up to 10 posts/day
- Data retention: store indefinitely
- Availability target: 99.9%
- Latency:
  - Create post ≤ 2 s
  - Get feed/list of posts ≤ 1 s
- Limits and constraints
  - Up to 5 images per post
  - Max image size: 3 MB
  - Text length: up to 1,000 characters
  - Comment length: up to 500 characters

## One post size (approx)
```text
Model:
  ID               8 B
  USER_ID          8 B
  PLACE_ID         8 B
  TEXT             ~2 KiB
  IMAGES           ~15 MiB
  COMMENT_IDs      ~20B
  LIKE_IDs         ~30B
  IMAGES PREVIEW   ~600 KiB
-----------------------------
Total              ≈ 15.7 MiB
```

## Basic sizing and back‑of‑the‑envelope calculations
Given:
- `DAU = 10,000,000`
- `avg_read_requests_per_user_per_day = 20`
- `avg_write_requests_per_user_per_day = 0.1`
- `max_images_per_post = 5`, `max_image_size = 3 MB` → worst‑case media per post `≈ 15 MB`
- Assume average read payload `≈ 602 KB` (1 post)
- `avg_likes_per_user_per_day = 10`
- `avg_comments_per_user_per_day = 1`


Derived:
- Read RPS: `10,000,000 × 20 / 86,400 ≈ 2 315 requests/s`
- Read traffic: `2,315 rps × 652 KB ≈ 1.5 GB/s`

- Write RPS:`10,000,000 × 0.1 / 86,400 ≈ 11.6 requests/s`
- Write ingress (media uploads): `11.6 rps × 15 MB ≈ 174 MB/s`

- Likes per day: `10,000,000 × 10 = 100,000,000` → Write RPS: `100,000,000 / 86,400 ≈ 1,157 rps`
- Comments per day: `10,000,000 × 1 = 10,000,000` → Write RPS: `10,000,000 / 86,400 ≈ 116 rps`

- Concurrent connections (assume 10% of DAU online simultaneously): `≈ 1,000,000`