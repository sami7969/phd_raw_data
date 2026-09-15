# phd_raw_data

Research data for a PhD project on social media news authenticity assessment: 35 Pakistani political
and news accounts on X (formerly Twitter), their posts, the comments on those posts, and the enrichment
values computed over them.

The repository holds two things:

| Path | What it is |
|---|---|
| `raw.zip` | The raw collection archive: 275 JSON files of posts and comments for the 35 accounts, as returned by the collection service, including post text and author objects. The collection pipeline is described in Shah, Mahoto and Shaikh, *Integrating Third-Party Aggregators for X (aka Twitter) Data Collection: A Pipeline Architecture* (Journal of Information Science, under review). |
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

Recomputable from the published columns (7 of the 18 catalog fields):
- **Engagement Ratio** (2A §3.3.2.4): the like, retweet, quote and reply counts, with the impression
  count as denominator.
- **Virality** (2B §3.2.1): the retweet count, with the components the deployment could not observe held
  as the specification sets them.
- **Per-Tweet Influence** (2B §3.2.2): the published Engagement Ratio, Retweet Velocity and Virality,
  with its author-standing term supplied as zero.
- **Engagement Patterns** (2B §3.3.1): post timestamps, interaction-type flags and reply counts. This
  holds for 22 of the 35 accounts. For the other 13, the stored value was computed over a seeded sample
  of 2,000 posts whose membership was not persisted.
- **Profile Influence** (2B §3.3.2): the published Per-Tweet Influence, Engagement Patterns and
  Cross-Reference values and the BERTopic assignments, with its network centrality input supplied as
  zero. This holds for 21 of the 35 accounts: not for the same 13 sampled accounts, nor for one account
  two of whose posts have no topic assignment.
- **Temporal Consistency** (2B §3.3.3): the topic labels, stance records, post timestamps and the
  approved alias rows.
- **Profile Credibility** (2B §3.3.5): the published Tweet Credibility, Cross-Reference support, Bot
  Likelihood, per-post mean Comment Affirmation, Toxicity, Temporal Consistency and Self-Contradiction
  values.

Not recomputable from the enriched release (11 of the 18):
- **Retweet Velocity** (2A §3.3.2.5): it divides the retweet count by the post's age at the moment
  enrichment ran, and that per-post enrichment time is not published.
- **Comment Direction** (2A §3.3.1.3): it reads the reply target recorded on each comment. That is the
  omitted `in_reply_to_user_id` column, corrupted on 16,964 rows.
- **The text- and model-based fields**, which apply models or rules to post or comment text that the
  enriched release does not contain:
  - Comment Sentiment, Comment Emotion and Comment Affirmation;
  - Tweet Sentiment, Tweet Emotion, Cross-Reference and Tweet Credibility;
  - Self-Contradiction, whose candidate pairs are verified by natural language inference and a
    language model.
- **Bot Likelihood** (2A §3.3.3.1), which is computed from eight behavioral signals that were not
  retained.

The raw collected records, including text, are in `raw.zip`. Translated text and intermediate model
outputs are not published.

**One caution on `topic`.** The `topic` column in `tweets` is a model-generated one-sentence label that a
language model derived from each post. It is not the post text, and it should not be read or quoted as
what the account wrote.

## Identifiers and joins

- `tweets.tweet_id` = `comments.parent_tweet_id` = `bertopic_topic_assignments.tweet_id`
- `tweets.author_id` = `profiles.user_id` for the posts of the 35 profiles
- `entity_aliases.surface_form` / `surface_norm` resolve the entity names found in `tweets.entities_json`
  and `tweets.stance_json` to `canonical_id`; only rows with `review_status = approved` are applied
  (all 297 published rows are approved)

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
| `enriched_data/data_dictionary.csv`, `.json` | | | | | as is |

**Row populations.** `tweets`: every enriched post (159,537). `comments`: the 876,857 enriched comments (135,275 real, 741,582 synthetic). `profiles`: the 35 accounts. `reference/entity_aliases`: the full curated alias table (297 rows, with review status). `reference/bertopic_topic_assignments`: the BERTopic group of each post (159,535 rows).

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
| `reply_count` | integer | [0, inf) | 0 to 33898 | 0 | 2A §3.3.2.4; 2B §3.3.1 | Reply count stored with the collected record; an Engagement Ratio input, and the reply counts Engagement Patterns reads. |
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
