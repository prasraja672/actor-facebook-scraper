# Facebook Page Crawler

Extract public information from Facebook Pages.

- [Features](#features)
- [Usage](#usage)
- [Input](#input)
- [Output](#output)
- [Expected Consumption](#expected-consumption)
- [Displaying only posts without page information](#displaying-only-posts-without-page-information)
- [Limitations / Caveats](#limitations--caveats)

## Features

* Extract page content and optionally filter them by minimum and maximum dates:
  * Scrape pages' posts
  * Scrape posts' comments
  * Scrape pages' reviews
* Get all page information, including:
  * Likes
  * Address (includes latitude / longitude)
  * Instagram profile
  * Twitter profile
  * Website
  * Services
  * Messenger url
  * Telephone
  * Check-ins
  * All other provided text information like awards, price range, mission
* Fetch businesses from the directory on https://www.facebook.com/biz/directory/

## Usage

If you want to run the actor on the Apify platform, you need to have at least some Apify proxy group so that Facebook doesn't block you. Since it uses Puppeteer, the minimum memory for running is 2048 MB.

## Input

Example input, only `startUrls` and `proxyConfiguration` are required (check `INPUT_SCHEMA.json` for settings):

```jsonc
{
    "startUrls": [
        { "url": "https://www.facebook.com/apifytech" },
        { "url": "https://www.facebook.com/biz/hotel-supply-service/?place_id=103095856397524" }
    ],
    "language": "en-US",
    "commentsMode": "RANKED_THREADED", // ["RANKED_THREADED", "RECENT_ACTIVITY", "RANKED_UNFILTERED"]
    "maxPosts": 3,
    "maxPostDate": "3 days", // or a static date in ISO format, like 2020-01-01
    "minPostDate": "1 day", // or statis date in ISO format
    "maxPostComments": 15,
    "maxCommentDate": "2020-01-01",
    "maxReviews": 3,
    "maxReviewDate": "2020-01-01",
    "scrapeAbout": true,
    "scrapeReviews": true,
    "scrapePosts": true,
    "scrapeServices": true,
    "proxyConfiguration": {
        "useApifyProxy": true,
        "apifyProxyGroups": ["SHADER"]
    }
}
```

## Output

```jsonc
{
    "categories": ["Hotel"],
    "info": [
        "Residenc", // ...
        "General Information\n" // ...
    ],
    "likes": 1538,
    "messenger": "https://m.me/22163", // ...
    "posts": [
        {
            "postDate": "2020-09-10T09:33:43.000Z",
            "postText": "Do Prahy opět", // ...
            "postImages": [
                {
                    "link": "https://www.facebook.com/Residen", //...
                    "image": "https://scontent-ort2-1.xx.fbcdn.net/v/t1.0" // ...
                }
            ],
            "postLinks": ["https://residen"], // ...
            "postUrl": "https://www.facebook.com/permalink.php?story_fbid=", // ...
            "postStats": {
                "comments": 1,
                "reactions": 32,
                "reactionsBreakdown": {
                    "like": 26,
                    "love": 6
                },
                "shares": 1
            },
            "postComments": {
                "count": 0,
                "mode": "RANKED_UNFILTERED",
                "comments": []
            }
        }
    ],
    "priceRange": "$$$",
    "title": "Hotel Resid", // ...
    "pageUrl": "https://www.facebook.com/Residen", //...
    "address": {
        "city": "Prague, Czech Republic",
        "lat": 50.09136,
        "lng": 14.42575,
        "postalCode": "11000",
        "region": "Prague",
        "street": "Haštalská 19"
    },
    "awards": [],
    "email": "", //...
    "impressum": [],
    "instagram": "@Residen", // ...
    "phone": "+420 22", //...
    "products": [],
    "transit": null,
    "twitter": "@Residen", //...
    "website": "http://", //...
    "youtube": null,
    "mission": [],
    "overview": [],
    "payment": null,
    "checkins": "2,082 people checked in here",
    "verified": false,
}
```

## Expected Consumption

One page and posts take around 1-2 minutes for the default amount of information (3 posts, 15 comments) to be generated, also depends on the proxy type used (`RESIDENTIAL` vs `DATACENTER`), block rate, retries, memory and CPU provided.

Usually, more concurrency is not better, while 5-10 concurrent tasks can finish each around 30-60s. A "20-concurrency" run can take up to 300s each. You can limit your concurrency by setting the `MAX_CONCURRENCY` environment variable on your actor.

A 2048MB actor takes an average `0.015` CU for each page on default settings. More "input page URLs" means more memory needed to scrape all pages.

**WARNING**: Don't use a limit too high for `maxPosts` as you can lose everything due to out of memory, or it may never finish. While scrolling the page, the partial content is kept in memory until the scrolling finishes.

Take into account the need for proxies that are included in the costs.

## Displaying only posts without page information

You can use the `unwind` parameter to display only the posts from your dataset on the platform, as such:

```
https://api.apify.com/v2/datasets/zbg3vVF3NnXGZfdsX/items?format=json&clean=1&unwind=posts&fields=posts,title,pageUrl
```

`unwind` will turn the `posts` property on the dataset to become the dataset items themselves. the `fields` parameters makes sure to only include the fields that are important

## Limitations / Caveats

* Personal profiles and groups aren't accessible at the moment
* Pages "Likes" count is a best-effort. The mobile page doesn't provide the count, and some languages don't provide any at all. So if a page has 1.9M, the number will most likely be 1900000 instead of the exact number.
* No content, stats or comments for live stream posts
* New reviews don't contain a rating from 1 to 5, but rather is positive or negative
* Cut-off date for posts happen on the original posted date, not edited date, i.e: posts show as `February 20th 2:11AM`, but that's the edited date, the actual post date is `February 19th 11:31AM` provided on the DOM
* The order of items aren't necessarily the same as seen on the page, and not sorted by date
* Comments of comments (nested comments / conversations) aren't included in the output, only top-level comments on the posts.

## Versioning

This project adheres to semver.

* Major versions means a change in the output or input format, and change in behavior.
* Minor versions mean new features
* Patch versions mean bug fixes/optimizations (changes to `README.md` aren't tagged)

## Upcoming

* Separated mode for posts, comments, and reviews (breaking change)
* Public groups

## License

Apache-2.0
