# phd_raw_data

Research data for a PhD project on social media news authenticity assessment: 35 Pakistani political
and news accounts on X (formerly Twitter), their posts, the comments on those posts, and the enrichment
values computed over them.

The repository holds two things:

| Path | What it is |
|---|---|
| `raw.zip` | The raw collection archive: 275 JSON files of posts and comments for the 35 accounts, as returned by the collection service, including post text and author objects. The collection pipeline is described in Shah, Mahoto and Shaikh, *Integrating Third-Party Aggregators for X (aka Twitter) Data Collection: A Pipeline Architecture* (under review). |
| `enriched_data/` | The enriched release (this README's subject): record identifiers plus the enrichment values that Papers 2A and 2B specify and evaluate, plus the supporting inputs and reference tables needed to re-derive them. It contains no post or comment text. |

The implementation code is not publicly released.

## How the release maps to the two papers

- **Paper 2A** specifies the eleven foundational enrichment fields: Comment Sentiment, Comment Emotion,
  Comment Direction and Comment Affirmation (comment level); Tweet Sentiment, Tweet Emotion,
  Cross-Reference, Tweet Credibility, Engagement Ratio and Retweet Velocity (tweet level); and Bot
  Likelihood (profile level).
- **Paper 2B** specifies the seven behavioral fields: Virality and Per-Tweet Influence (tweet level);
  Engagement Patterns, Profile Influence, Temporal Consistency, Self-Contradiction and Profile
  Credibility (profile level). Its Table 3 lists the supporting inputs those fields consume that are
  not catalog fields: the tweet topic label, entity stance, extracted entities, toxicity, the entity
  alias mapping and the BERTopic grouping.

Together these are the eighteen catalog fields. The stored value of every catalog field is in
`enriched_data/`, together with every supporting input that Paper 2B's Table 3 names. The data dictionary
below names, for each column, the section of the paper that defines it.

**What the enriched release is enough to recompute, and what it is not.** Each field's inputs are as its
defining section states. Where a stored value itself cannot be rebuilt from persisted inputs, Paper 2B's
reproducibility table (Section 5.2) says so, and that limit is repeated here.

Recomputable from the published columns (8 of the 18 catalog fields):
- **Engagement Ratio** (2A §3.3.2.4): the like, retweet, quote and reply counts, with the impression
  count as denominator.
- **Virality** (2B §3.2.1): the retweet count, with the components the deployment could not observe held
  as the specification sets them.
- **Per-Tweet Influence** (2B §3.2.2): the published Engagement Ratio, Retweet Velocity and Virality,
  with its author-standing term supplied as zero.
- **Engagement Patterns** (2B §3.3.1): post timestamps and interaction-type flags; its outgoing reply
  count is the number of posts with `is_reply = 1`, and incoming replies enter as zero. It does not read
  `reply_count`. This holds for 28 of the 35 accounts. For 13 accounts the stored value was computed
  over a seeded sample of 2,000 posts, whose membership the run did not persist. For 6 of them
  (HamidMirPAK, HassanNisar, MoeedNj, OryaMaqboolJan, javeednusrat, mubasherlucman) no two posts share
  a `created_at`, so the sample is fully determined by the published timestamps; it is regenerated
  and published in `eps_pi_samples/sampled_posts_2000`, and replaying the field over it from this
  release alone reproduces every stored value exactly. For the other 7, tied `created_at` values
  leave the draw order unrecorded, so the sample cannot be regenerated.
- **Profile Influence** (2B §3.3.2): the published Per-Tweet Influence, Engagement Patterns and
  Cross-Reference values and the BERTopic assignments, with its network centrality input supplied as
  zero. This holds for 27 of the 35 accounts, including the same 6 sampled accounts over their
  published samples (exact): not for the other 7 sampled accounts, nor for one account two of whose
  posts have no topic assignment.
- **Temporal Consistency** (2B §3.3.3): the topic labels, stance records, post timestamps and the
  approved alias rows.
- **Self-Contradiction** (2B §3.3.4): for all 35 accounts, from the retained pairs in
  `self_contradiction/self_contradiction_pairs` (each pair's weight `w_i` and contradiction probability
  `p_i`) and each account's enriched post count `N` in `self_contradiction/self_contradiction_accounts`:
  SC = clamp((Σ w_i p_i / Σ w_i)(Σ w_i / N), 0, 1), summed in the published row order, reproduces every
  stored `contradiction_score` exactly. This recomputes the stored value from the retained pairs; it
  does not rebuild the pairs, which were selected by natural language inference and language-model
  verification over translated text that is not published. For the 17 accounts with more than 1,000
  posts, `self_contradiction/sampled_posts` lists the 1,000 posts candidate construction drew from.
- **Profile Credibility** (2B §3.3.5): the published Tweet Credibility, Cross-Reference support, Bot
  Likelihood, per-post mean Comment Affirmation, Toxicity, Temporal Consistency and Self-Contradiction
  values.

Not recomputable from the enriched release (10 of the 18):
- **Retweet Velocity** (2A §3.3.2.5): it divides the retweet count by the post's age at the moment
  enrichment ran, and that per-post enrichment time is not published.
- **Comment Direction** (2A §3.3.1.3): it reads the reply target recorded on each comment. That is the
  omitted `in_reply_to_user_id` column, corrupted on 16,964 rows.
- **The text- and model-based fields**, which apply models or rules to post or comment text that the
  enriched release does not contain:
  - Comment Sentiment, Comment Emotion and Comment Affirmation;
  - Tweet Sentiment, Tweet Emotion, Cross-Reference and Tweet Credibility.
- **Bot Likelihood** (2A §3.3.3.1), which is computed from eight behavioral signals that were not
  retained.

The raw collected records, including text, are in `raw.zip`. Translated text is not published, and
neither are intermediate model outputs, except the contradiction probability of each retained
Self-Contradiction pair.

### Input map for the seven behavioral fields (Paper 2B)

Each input of each behavioral field, its kind, and the file and column that carry it. Kinds:
**platform metadata** (collected with the record), **foundational field** (Paper 2A), **behavioral
field** (Paper 2B, consumed by a later field), **supporting measurement** (Paper 2B Table 3), and
**reference resource** (a separate table), and **intermediate model output** (the BERTopic grouping, Paper 2B Table 3). Files are under `enriched_data/`.

| Field | Input | Kind | File : column |
|---|---|---|---|
| Virality | retweet count | platform metadata | `tweets` : `retweet_count` |
| Per-Tweet Influence | Engagement Ratio | foundational field | `tweets` : `engagement_ratio` |
| | Retweet Velocity | foundational field | `tweets` : `retweet_velocity` |
| | Virality | behavioral field | `tweets` : `virality_score` |
| Engagement Patterns | post timestamps | platform metadata | `tweets` : `created_at` |
| | seeded 2,000-post sample (6 of the 13 sampled accounts) | — | `eps_pi_samples/sampled_posts_2000` : `tweet_id` |
| | interaction type (outgoing replies are the posts with `is_reply = 1`) | platform metadata | `tweets` : `is_reply`, `is_quote`, `is_retweet` |
| Profile Influence | Per-Tweet Influence | behavioral field | `tweets` : `tweet_influence_score` |
| | Cross-Reference support (polarization proxy) | foundational field | `tweets` : `cross_reference_score` |
| | BERTopic grouping | intermediate model output | `reference/bertopic_topic_assignments` : `topic_id` |
| | Engagement Patterns | behavioral field | `profiles` : `engagement_patterns_score` |
| | seeded 2,000-post sample (the same 6 accounts) | — | `eps_pi_samples/sampled_posts_2000` : `tweet_id` |
| Temporal Consistency | topic label | supporting measurement | `tweets` : `topic` |
| | entity stance | supporting measurement | `tweets` : `stance_json` |
| | post timestamps | platform metadata | `tweets` : `created_at` |
| | entity aliases (approved rows) | reference resource | `reference/entity_aliases` : `surface_norm`, `canonical_id`, `review_status` |
| Self-Contradiction | retained pairs, weight and contradiction probability | behavioral field (intermediate) | `self_contradiction/self_contradiction_pairs` : `tweet_a_id`, `tweet_b_id`, `w_i`, `p_i` |
| | enriched post count (denominator) | platform metadata | `self_contradiction/self_contradiction_accounts` : `N` (equals the account's rows in `tweets`) |
| | candidate groups and pair weighting | platform metadata; supporting measurement; intermediate model output; reference resource | `tweets` : `mentioned_handles`, `entities_json`; `reference/bertopic_topic_assignments` : `topic_id`; `reference/entity_aliases` |
| | seeded sample (17 accounts) | — | `self_contradiction/sampled_posts` : `tweet_id` |
| Profile Credibility | Tweet Credibility | foundational field | `tweets` : `tweet_credibility_score` |
| | Cross-Reference support | foundational field | `tweets` : `cross_reference_score` |
| | Comment Affirmation, per-post mean | foundational field | `tweets` : `avg_affirmation_score` |
| | toxicity | supporting measurement | `tweets` : `toxicity` |
| | Temporal Consistency, Self-Contradiction | behavioral field | `profiles` : `consistency_score`, `contradiction_score` |
| | Bot Likelihood | foundational field | `profiles` : `is_bot` |

Inputs the specifications name that the deployment supplied as zero (Virality's cascade depth,
branching and half-life; Per-Tweet Influence's author standing; Engagement Patterns' incoming replies;
Profile Influence's network centrality) have no column. Not published: the membership of the 2,000-post
seeded samples over which Engagement Patterns and Profile Influence were computed for 7 accounts (the
samples of the other 6 sampled accounts are in `eps_pi_samples/sampled_posts_2000`); the topic
assignment of two posts of one account; and the translated post text.

**One caution on `topic`.** The `topic` column in `tweets` is a model-generated one-sentence label that a
language model derived from each post. It is not the post text, and it should not be read or quoted as
what the account wrote.

## Identifiers and joins

- `tweets.tweet_id` = `comments.parent_tweet_id` = `bertopic_topic_assignments.tweet_id`
- `tweets.author_id` = `profiles.user_id` for the posts of the 35 profiles
- `entity_aliases.surface_form` / `surface_norm` resolve the entity names found in `tweets.entities_json`
  and `tweets.stance_json` to `canonical_id`; only rows with `review_status = approved` are applied
  (all 297 published rows are approved)
- `self_contradiction_pairs.tweet_a_id` / `tweet_b_id` = `tweets.tweet_id`; `sampled_posts.tweet_id` = `tweets.tweet_id`;
  `self_contradiction_pairs.handle` = `self_contradiction_accounts.handle` = `profiles.handle`
- `eps_pi_samples/sampled_posts_2000.tweet_id` = `tweets.tweet_id`; its `handle` = `profiles.handle`

Identifiers are strings, because X ids exceed the range a double-precision number holds exactly.

## Synthetic comments

`comments.is_synthetic` is 1 for **741,582** comments that were generated with a large language model,
to augment sparse threads and to provide a controlled testbed in which the intended value of each field
is known in advance, and 0 for the **135,275** real collected comments in the release. Synthetic
comments carry the handle `syn_user_N` and no `created_at`. No synthetic comment is presented as, or
attributable to, a real user. Nineteen of the 35 profiles have no real comments, so comment-derived
values for those profiles describe generated text; Paper 2A reports distributions for the real and
synthetic subsets separately and measures their fidelity (Section 5.5).

## Files

| File | Rows | Columns | Format | Size | Published as |
|---|--:|--:|---|--:|---|
| `enriched_data/tweets.csv` | 159,537 | 28 | CSV | 143.2 MB | `tweets.csv.zip.001` (1 part, zip 41.7 MB) |
| `enriched_data/tweets.json` | 159,537 | 28 | JSON | 215.3 MB | `tweets.json.zip.001` (1 part, zip 44.5 MB) |
| `enriched_data/comments.csv` | 876,857 | 11 | CSV | 400.7 MB | `comments.csv.zip.001` to `comments.csv.zip.004` (4 parts, zip 151.4 MB) |
| `enriched_data/comments.json` | 876,857 | 11 | JSON | 588.7 MB | `comments.json.zip.001` to `comments.json.zip.004` (4 parts, zip 160.0 MB) |
| `enriched_data/profiles.csv` | 35 | 8 | CSV | 5.5 KB | as is |
| `enriched_data/profiles.json` | 35 | 8 | JSON | 9.8 KB | as is |
| `enriched_data/reference/entity_aliases.csv` | 297 | 12 | CSV | 58.9 KB | as is |
| `enriched_data/reference/entity_aliases.json` | 297 | 12 | JSON | 105.8 KB | as is |
| `enriched_data/reference/bertopic_topic_assignments.csv` | 159,535 | 4 | CSV | 10.8 MB | as is |
| `enriched_data/reference/bertopic_topic_assignments.json` | 159,535 | 4 | JSON | 18.7 MB | as is |
| `enriched_data/self_contradiction/self_contradiction_pairs.csv` | 513 | 10 | CSV | 65.1 KB | as is |
| `enriched_data/self_contradiction/self_contradiction_pairs.json` | 513 | 10 | JSON | 119.1 KB | as is |
| `enriched_data/self_contradiction/self_contradiction_accounts.csv` | 35 | 6 | CSV | 1.8 KB | as is |
| `enriched_data/self_contradiction/self_contradiction_accounts.json` | 35 | 6 | JSON | 4.0 KB | as is |
| `enriched_data/self_contradiction/sampled_posts.csv` | 17,000 | 2 | CSV | 606.5 KB | as is |
| `enriched_data/self_contradiction/sampled_posts.json` | 17,000 | 2 | JSON | 988.3 KB | as is |
| `enriched_data/eps_pi_samples/sampled_posts_2000.csv` | 12,000 | 2 | CSV | 427.8 KB | as is |
| `enriched_data/eps_pi_samples/sampled_posts_2000.json` | 12,000 | 2 | JSON | 697.3 KB | as is |
| `enriched_data/data_dictionary.csv`, `.json` | | | | | as is |

**Row populations.** `tweets`: every enriched post (159,537). `comments`: the 876,857 enriched comments (135,275 real, 741,582 synthetic). `profiles`: the 35 accounts. `reference/entity_aliases`: the full curated alias table (297 rows, with review status). `reference/bertopic_topic_assignments`: the BERTopic group of each post (159,535 rows). `self_contradiction/self_contradiction_pairs`: the 513 retained Self-Contradiction pairs, 21 of them flagged `content_free`. `self_contradiction/self_contradiction_accounts`: the 35 accounts, 17 of them sample-qualified. `self_contradiction/sampled_posts`: the 1,000-post seeded sample of each of those 17 accounts. `eps_pi_samples/sampled_posts_2000`: the 2,000-post seeded sample of Engagement Patterns and Profile Influence for the 6 of the 13 sampled accounts whose sample can be regenerated.

## What is not in the enriched release, and why

| File | Omitted | Reason |
|---|---|---|
| tweets | text, translated_text | post text is not part of the enriched release |
| tweets | claim_type, aspect_sentiment_json | model outputs that no field specified in either paper consumes |
| tweets | retweeted_tweet_id, quoted_tweet_id | NULL on every row |
| tweets | language, hashtags, urls, media_urls, collected_at, comment_count, enriched_at | collection and enrichment-run metadata outside the enriched release |
| comments | text, translated_text | comment text is not part of the enriched release |
| comments | in_reply_to_user_id | stored identifiers are corrupted on 16,964 rows by a floating-point conversion at ingest; publishing them would be worse than omitting the column. Comment Direction reads this reply target, so it cannot be recomputed from the enriched release |
| comments | mentioned_handles | no field specified in either paper consumes them at comment level |
| comments | language, like_count, reply_count, collected_at, enriched_at | collection metadata and platform counts outside the enriched release |
| comments | 17,203 raw comments (rows) | real comments whose parent post is not in the collected post table, so no enrichment values were computed for them |
| profiles | display_name, description, location, profile_image_url | profile text and media are not part of the enriched release |
| profiles | followers_count, following_count, tweet_count, listed_count, verified, created_at, collected_at | collection metadata outside the enriched release |
| profiles | consistency_detail, contradiction_detail, tweet_count_enriched, avg_tweet_credibility, avg_tweet_influence, fib_index, cib_score, cib_evidence_json, agenda_score, agenda_evidence_json, enriched_at | diagnostic detail and analysis outputs that are not catalog fields |

Post and comment text is excluded from the enriched release. Note that `raw.zip` in this repository does contain the raw collected records, including text.

## Reassembling the split files

Files larger than GitHub's 100 MB file limit are zipped and split into parts of at most 49 MB, named
`<file>.zip.001`, `<file>.zip.002`, and so on. The parts are ordinary files (not Git LFS). To restore a
file, concatenate its parts in order, check the archive's checksum against
`SHA256SUMS_reassembled.txt`, and unzip. Every part's checksum is in `SHA256SUMS.txt`.

**bash (Linux, macOS, Git Bash)**, from inside `enriched_data/`:

```bash
sha256sum -c SHA256SUMS.txt                       # every published part and file
cat comments.csv.zip.* > comments.csv.zip          # parts sort in order: .001, .002, ...
sha256sum comments.csv.zip                         # compare with SHA256SUMS_reassembled.txt
unzip comments.csv.zip                             # gives comments.csv; its checksum is also listed
```

**PowerShell (Windows)**, from inside `enriched_data`:

```powershell
Get-FileHash -Algorithm SHA256 comments.csv.zip.* | Format-Table Hash, Path
$parts = (Get-ChildItem comments.csv.zip.* | Sort-Object Name | ForEach-Object { $_.Name }) -join '+'
cmd /c copy /b $parts comments.csv.zip
Get-FileHash -Algorithm SHA256 comments.csv.zip    # compare with SHA256SUMS_reassembled.txt
Expand-Archive comments.csv.zip -DestinationPath .
```

Replace `comments.csv` with the file you need; the table above lists each file's parts.

## File formats

- **CSV:** UTF-8, LF line endings, one header row. Every non-null value is quoted; a null is an empty
  unquoted field, and an empty string is `""`. Real numbers use the shortest representation that reads
  back to the identical stored double.
- **JSON:** a UTF-8 JSON array with one record object per line; nulls are `null`.
- Columns stored in the database as JSON (the sentiment, emotion and cross-reference vectors,
  `image_sentiment`, `mentioned_handles`, `stance_json`, `entities_json`) are published in both formats
  as the stored JSON string, byte for byte. Parse the string to get the array.

## Data dictionary

Also published as `enriched_data/data_dictionary.csv` and `.json`. Sections: **2A** = Paper 2A, **2B** = Paper 2B (citations below). *Observed* is computed from the published values.

### `tweets`

| Column | Type | Range | Observed | Nulls | Defined in | Description |
|---|---|---|---|--:|---|---|
| `tweet_id` | string (digits) | platform post id |  | 0 | 2A §5.1; 2B §5.1 | X post identifier. Primary key; joins comments.parent_tweet_id and bertopic_topic_assignments.tweet_id. |
| `author_id` | string (digits) | platform user id |  | 0 | 2A §5.1 | X user identifier of the post's author; joins profiles.user_id for the 35 profiles. |
| `author_handle` | string | X handle without @ |  | 0 | 2A §5.1 | Handle of the post's author. 159,060 posts belong to the 35 profiles; 477 posts by 342 other accounts are in the collected post table. |
| `in_reply_to_tweet_id` | string (digits) or null | platform post id |  | 147,502 | 2A §5.1 | Post id this post replies to, where the collected record carried one. |
| `created_at` | string (ISO 8601, UTC offset) | timestamp |  | 0 | 2B §3.3.3 | Post creation time as collected. Temporal Consistency uses post timestamps. |
| `retweet_count` | integer | [0, inf) | 0 to 59227 | 0 | 2A §3.3.2.4; 2A §3.3.2.5; 2B §3.2.1 | Retweet count stored with the collected record. Engagement Ratio, Retweet Velocity and Virality's reach component read it. |
| `like_count` | integer | [0, inf) | 0 to 150598 | 0 | 2A §3.3.2.4 | Like count stored with the collected record; an Engagement Ratio input. |
| `reply_count` | integer | [0, inf) | 0 to 33898 | 0 | 2A §3.3.2.4 | Reply count stored with the collected record; an Engagement Ratio input. Engagement Patterns does not read it: its outgoing reply count is the number of posts with `is_reply = 1`. |
| `quote_count` | integer | [0, inf) | 0 to 4202 | 0 | 2A §3.3.2.4 | Quote count stored with the collected record; an Engagement Ratio input. |
| `view_count` | integer | [0, inf); 0 where no impression count was available | 0 to 9989511 | 0 | 2A §3.3.2.4; 2B §5.3 | Impression count stored with the collected record; the Engagement Ratio denominator is this count plus one. It is 0 on the 73,840 posts without an impression count, whose stored Engagement Ratio is therefore the unnormalized weighted interaction count. |
| `is_reply` | integer | 0 or 1 | 0 to 1 | 0 | 2B §3.3.1 | Interaction-type flag as collected; Engagement Patterns' interaction diversity reads the interaction type of each post. |
| `is_quote` | integer | 0 or 1 | 0 to 1 | 0 | 2B §3.3.1 | Interaction-type flag as collected (see is_reply). |
| `is_retweet` | integer | 0 or 1 | 0 to 0 | 0 | 2B §3.3.1 | Interaction-type flag as collected (see is_reply). |
| `tweet_sentiment_score` | string (JSON array of 6 reals) | each in [0, 1] | length 6 | 0 | 2A §3.3.2.1 | Tweet Sentiment: positive, negative, neutral, mixed, ambiguity (stored as objective), sarcasm. |
| `tweet_emotion_score` | string (JSON array of 14 reals) | each in [0, 1] | length 14 | 0 | 2A §3.3.2.1 | Tweet Emotion: joy, sadness, anger, fear, disgust, surprise, trust, anticipation, love, optimism, pessimism, remorse, pride, curiosity. Not normalized. |
| `tweet_credibility_score` | real | [0, 1] | 0.004285714285714285 to 1.0 | 0 | 2A §3.3.2.3 | Tweet Credibility. |
| `cross_reference_score` | string (JSON array of 3 reals) | each in [0, 1] | length 3 | 0 | 2A §3.3.2.2 | Cross-Reference verdict vector: support, contradict, unverified. |
| `engagement_ratio` | real | [0, inf) | 0.0 to 202154.5 | 0 | 2A §3.3.2.4; 2B §5.3 | Engagement Ratio: weighted interaction count over impressions plus one. Where no impression count was available (73,840 posts) the stored value is the unnormalized weighted interaction count. |
| `retweet_velocity` | real | [0, inf) | 0.0 to 316.5323424777606 | 0 | 2A §3.3.2.5 | Retweet Velocity. |
| `virality_score` | real | declared [0, 1]; deployed [0.200, 0.600] | 0.2 to 0.6000000000000001 | 0 | 2B §3.2.1; 2B §5.5 | Virality. |
| `tweet_influence_score` | real | declared [0, 1]; deployed [0.070, 0.760] | 0.06999999999999999 to 0.5616360800389444 | 0 | 2B §3.2.2; 2B §5.5 | Per-Tweet Influence. |
| `cross_reference_confidence` | real | [0, 1] | 0.0 to 0.3934693402873666 | 0 | 2A §3.3.2.2 | Evidence-retrieval indicator attached to the Cross-Reference verdict: a retrieval-coverage measure, not a calibrated confidence in the verdict's correctness. Greater than zero on the 58,944 posts that received a verdict. |
| `avg_affirmation_score` | real | [-1, 1] | -0.6277404837302871 to 0.7273130010422038 | 0 | 2A §3.3.1.4; 2B §3.3.5 | Mean Comment Affirmation over the post's enriched comments. Profile Credibility's crowd component consumes the account mean of this value. |
| `mentioned_handles` | string (JSON array of handles) | [] when none | 136,175 empty | 0 | 2B §3.3.4 | Handles mentioned in the post, as collected. Self-Contradiction forms candidate pairs within mention-handle groups and weights pairs by mention-group membership. |
| `topic` | string or null | one sentence |  | 5,214 | 2B Table 3; 2B §3.3.3 | Tweet topic label: a MODEL-GENERATED one-sentence label derived from the post by a language model. It is not the post text. Temporal Consistency groups posts by it. |
| `stance_json` | string (JSON array of {entity, stance, confidence}) | stance support/against/neutral; confidence [0, 1] | 36,583 empty | 0 | 2B Table 3; 2B §3.3.3 | Per-entity stance records. A small number of stored labels fall outside support/against/neutral; Temporal Consistency maps them to neutral. |
| `entities_json` | string (JSON array of {name, type, sub_type, party}) | entity records | 38,115 empty | 0 | 2B Table 3; 2B §3.3.4 | Extracted named entities. Self-Contradiction weights a pair fully when its posts share an alias-resolved entity. Stored type labels are as extracted, including some outside the intended type list. |
| `toxicity` | real | [0, 1] | 0.0 to 1.0 | 0 | 2B Table 3; 2B §3.3.5 | Tweet-level toxicity. Profile Credibility consumes the account mean through two paths (content complement and crowd multiplier). |

### `comments`

| Column | Type | Range | Observed | Nulls | Defined in | Description |
|---|---|---|---|--:|---|---|
| `comment_id` | string (digits) | platform or synthetic comment id |  | 0 | 2A §5.1 | Comment identifier. Primary key. |
| `parent_tweet_id` | string (digits) | platform post id |  | 0 | 2A §5.1 | The post the comment belongs to; joins tweets.tweet_id (every exported comment has its parent in tweets). |
| `author_id` | string | platform user id, or a synthetic id |  | 0 | 2A §5.1 | Identifier of the comment's author. |
| `author_handle` | string | X handle without @; syn_user_N for synthetic |  | 0 | 2A §5.1 | Handle of the comment's author. Synthetic comments carry syn_user_N. 67 real comments carry an empty handle as collected. |
| `is_synthetic` | integer | 0 or 1 | 0 to 1 | 0 | 2A Ethics statement; 2A §5.5 | 1 for the 741,582 comments generated with a large language model; 0 for real collected comments. |
| `created_at` | string (ISO 8601, UTC offset) or null | timestamp |  | 741,582 | 2A §5.1 | Comment creation time as collected. NULL on every synthetic comment, and only there: synthetic comments carry no timestamp. |
| `comment_sentiment_score` | string (JSON array of 6 reals) | each in [0, 1] | length 6 | 0 | 2A §3.3.1.1 | Comment Sentiment: positive, negative, neutral, mixed, ambiguity (stored as objective), sarcasm. |
| `comment_emotion_score` | string (JSON array of 14 reals) | each in [0, 1] | length 14 | 0 | 2A §3.3.1.2 | Comment Emotion, same fourteen dimensions as tweet_emotion_score. Not normalized. |
| `comment_affirmation_score` | real | [-1, 1] | -0.8812177393084712 to 0.8550802473552193 | 0 | 2A §3.3.1.4 | Comment Affirmation. |
| `comment_direction_type` | string | author, peer, external | values author, external, peer | 0 | 2A §3.3.1.3 | Comment Direction class. |
| `image_sentiment` | string (JSON array of 5 reals) or null | each in [0, 1] | length 5 | 876,128 | 2A §3.3.1.1 | Image classification scores (sarcastic_meme, mockery, hostile, supportive, informational) behind the 729 comment sentiment vectors that carry an image-derived adjustment, the exception 2A §3.3.1.1 states. All 729 are synthetic comments. NULL elsewhere. |

### `profiles`

| Column | Type | Range | Observed | Nulls | Defined in | Description |
|---|---|---|---|--:|---|---|
| `user_id` | string (digits) | platform user id |  | 0 | 2A §5.1 | X user identifier. Primary key; joins tweets.author_id. |
| `handle` | string | X handle without @ |  | 0 | 2A §5.1 | Handle of the profile. |
| `is_bot` | real | [0, 1] | 0.0015392826002223654 to 0.2773127775450356 | 0 | 2A §3.3.3.1 | Bot Likelihood. |
| `engagement_patterns_score` | real | declared [0, 1]; deployed [0.0119, 0.9363] | 0.125 to 0.8951859542009739 | 0 | 2B §3.3.1; 2B §5.5 | Engagement Patterns. |
| `influence_score` | real | declared [0, 0.79]; deployed [0.0373713, 0.687253] | 0.0781065700982928 to 0.3944510985104028 | 0 | 2B §3.3.2; 2B §5.5 | Profile Influence. |
| `consistency_score` | real | [0, 1] | 0.8738977072310405 to 1.0 | 0 | 2B §3.3.3 | Temporal Consistency. |
| `contradiction_score` | real | [0, 1] | 0.0 to 0.09394301418904905 | 0 | 2B §3.3.4 | Self-Contradiction. |
| `credibility_score` | real | [0, 1] | 0.4201974117905082 to 0.6652351354942609 | 0 | 2B §3.3.5 | Profile Credibility. |

### `reference/entity_aliases`

| Column | Type | Range | Observed | Nulls | Defined in | Description |
|---|---|---|---|--:|---|---|
| `alias_id` | integer | row id | 1 to 301 | 0 | 2B Table 3 | Alias row identifier. |
| `surface_form` | string |  |  | 0 | 2B Table 3 | Surface form of an entity name as it occurs in extracted entities or stance records. |
| `surface_norm` | string |  |  | 0 | 2B Table 3 | Normalized surface form used for matching. |
| `canonical_id` | string |  |  | 0 | 2B Table 3; 2B §3.3.3; 2B §3.3.4 | Canonical entity the surface form resolves to. |
| `canonical_name` | string |  |  | 0 | 2B Table 3 | Display name of the canonical entity. |
| `entity_type` | string | PERSON, ORGANIZATION, PARTY, LOCATION | values LOCATION, ORGANIZATION, PARTY, PERSON | 0 | 2B Table 3 | Entity type of the canonical entity. |
| `sub_type` | string or null |  | values city, country, government, international, journalist, judiciary, military, politician, province, region | 188 | 2B Table 3 | Optional finer type. |
| `source` | string | manual_seed, llm_cluster_qwen2.5_14b | values llm_cluster_qwen2.5_14b, manual_seed | 0 | 2B Table 3 | How the mapping was proposed. |
| `confidence` | real | [0, 1] | 0.9 to 1.0 | 0 | 2B Table 3 | Curator confidence in the mapping. |
| `review_status` | string | approved, unreviewed, rejected | values approved | 0 | 2B §3.3.3 | Only rows with review_status = approved are applied by Temporal Consistency and contradiction weighting. Every one of the 297 published rows is approved. |
| `notes` | string |  |  | 0 | 2B Table 3 | Curator note (for example a mention rank). |
| `created_at` | string (ISO 8601) | timestamp |  | 0 | 2B Table 3 | When the mapping row was created. |

### `reference/bertopic_topic_assignments`

| Column | Type | Range | Observed | Nulls | Defined in | Description |
|---|---|---|---|--:|---|---|
| `tweet_id` | string (digits) | platform post id |  | 0 | 2B Table 3 | Post identifier; joins tweets.tweet_id. Two posts of the tweet table have no assignment. |
| `topic_id` | integer | -1 (outlier) or 0..230 | -1 to 230 | 0 | 2B Table 3; 2B §3.3.2; 2B §3.3.4 | BERTopic group of the post. Profile Influence aggregates per group, and Self-Contradiction forms candidate pairs within groups. |
| `probability` | real | stored 0.0 on every row | 0.0 to 0.0 | 0 | 2B Table 3 | Assignment probability column as stored. It is 0.0 on all rows: no probability was recorded, and no field uses it. |
| `assigned_at` | string (ISO 8601) | timestamp | values 2026-05-04T10:24:25.877157+00:00 | 0 | 2B Table 3 | Time the assignments were written (one run). |

### `self_contradiction/self_contradiction_pairs`

| Column | Type | Range | Observed | Nulls | Defined in | Description |
|---|---|---|---|--:|---|---|
| `pair_id` | integer | row id | 14741 to 15253 | 0 | 2B §3.3.4 | Identifier of the retained pair as stored. Rows are in stored order, the order the score sums them in. |
| `handle` | string | X handle without @ |  | 0 | 2B §3.3.4 | Account whose two posts form the pair; joins profiles.handle. |
| `tweet_a_id` | string (digits) | platform post id |  | 0 | 2B §3.3.4 | First post of the pair; joins tweets.tweet_id. |
| `tweet_b_id` | string (digits) | platform post id |  | 0 | 2B §3.3.4 | Second post of the pair; joins tweets.tweet_id. |
| `group_type` | string | topic, entity | values entity (53), topic (460) | 0 | 2B §3.3.4 | Candidate group the pair was formed in: a BERTopic group (topic) or a mention-handle group (entity). |
| `group_key` | string |  |  | 0 | 2B §3.3.4 | The group: the BERTopic topic_id for a topic group, the mentioned handle for an entity group. |
| `w_i` | real | 0.5 or 1.0 | values 0.5 (298), 1.0 (215) | 0 | 2B §3.3.4 | Pair weight w_i: 1.0 when the pair was formed in a mention-handle group or its posts share an alias-resolved named entity (approved alias rows), 0.5 otherwise. |
| `p_i` | real | [0.75, 1] | 0.7500389814376831 to 0.9994400143623352 | 0 | 2B §3.3.4 | Contradiction probability p_con,i of the pair under the natural language inference model, as stored. |
| `entity_bucket` | string | entity, topic_overlap_ge_1, topic_overlap_0 | values entity (53), topic_overlap_0 (298), topic_overlap_ge_1 (162) | 0 | 2B §3.3.4 | Weighting rule that applied: entity (mention-handle group, 1.0), topic_overlap_ge_1 (shared entity, 1.0), topic_overlap_0 (no shared entity, 0.5). |
| `content_free` | integer | 0 or 1 | values 0 (492), 1 (21) | 0 | 2B §5.3; 2B §6 | 1 for the pairs whose two posts carry no words beyond a shortened link. They are included in the stored Self-Contradiction scores. |

### `self_contradiction/self_contradiction_accounts`

| Column | Type | Range | Observed | Nulls | Defined in | Description |
|---|---|---|---|--:|---|---|
| `handle` | string | X handle without @ |  | 0 | 2B §3.3.4 | Account; joins profiles.handle. |
| `user_id` | string (digits) | platform user id |  | 0 | 2B §3.3.4 | X user identifier; joins profiles.user_id. |
| `N` | integer | [0, inf) | 1 to 56613 | 0 | 2B §3.3.4 | N_enriched, the density denominator: the account's enriched posts (its rows in tweets). |
| `sample_size` | integer | min(N, 1000) | 1 to 1000 | 0 | 2B §3.3.4 | Posts candidate construction drew pairs from: all N posts, or a seeded sample of 1,000 when N exceeds 1,000. |
| `sample_qualified` | integer | 0 or 1 | values 0 (18), 1 (17) | 0 | 2B §5.3 | 1 when N exceeds 1,000, so pairs come from the seeded sample while the denominator is the full timeline. |
| `retained_pairs` | integer | [0, inf) | 0 to 53 | 0 | 2B §3.3.4 | Rows of self_contradiction_pairs for the account. |

### `self_contradiction/sampled_posts`

| Column | Type | Range | Observed | Nulls | Defined in | Description |
|---|---|---|---|--:|---|---|
| `handle` | string | X handle without @ | 17 accounts, 1,000 posts each | 0 | 2B §3.3.4 | Sample-qualified account; joins self_contradiction_accounts.handle. |
| `tweet_id` | string (digits) | platform post id |  | 0 | 2B §3.3.4 | A post in the account's seeded sample, regenerated as the run draws it (seed 42, 1,000 posts, ordered by created_at); the run did not record its sample. Joins tweets.tweet_id. |

### `eps_pi_samples/sampled_posts_2000`

| Column | Type | Range | Observed | Nulls | Defined in | Description |
|---|---|---|---|--:|---|---|
| `handle` | string | X handle without @ | 6 accounts, 2,000 posts each | 0 | 2B §3.3.1; 2B §3.3.2 | Sampled account; joins profiles.handle. The 6 of the 13 accounts above 2,000 posts whose posts have no tied created_at, so the run's draw is regenerated exactly. |
| `tweet_id` | string (digits) | platform post id |  | 0 | 2B §3.3.1; 2B §3.3.2 | A post in the account's seeded sample, regenerated as the run draws it (seed 42, 2,000 posts, ordered by created_at); Engagement Patterns and Profile Influence replayed over it reproduce the stored values exactly. Joins tweets.tweet_id. |

## Citation

If you use this data, please cite both papers:

```text
Shah, S. S., Mahoto, N. A., Shaikh, A., Awais, A. M., and Ameer, I. (2026).
An AI-Based Multi-Level Enrichment Approach for Social Media News Authenticity Assessment.
Under review.

Shah, S. S., Mahoto, N. A., and Shaikh, A. (2026).
Behavioral Enrichment Fields for Social Media Authenticity Assessment: Self-Contradiction,
Consistency, Influence, and Credibility. Under review.
```

```bibtex
@unpublished{shah2026enrichment,
  author = {Shah, Syed Samiullah and Mahoto, Naeem Ahmed and Shaikh, Asadullah and Awais, Agha Muhammad and Ameer, Iqra},
  title  = {An AI-Based Multi-Level Enrichment Approach for Social Media News Authenticity Assessment},
  note   = {Under review},
  year   = {2026}
}

@unpublished{shah2026behavioral,
  author = {Shah, Syed Samiullah and Mahoto, Naeem Ahmed and Shaikh, Asadullah},
  title  = {Behavioral Enrichment Fields for Social Media Authenticity Assessment: Self-Contradiction, Consistency, Influence, and Credibility},
  note   = {Under review},
  year   = {2026}
}
```
