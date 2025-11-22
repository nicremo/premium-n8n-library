# Ghost Automated Blog System

A fully automated content pipeline that runs hands-free every morning, generating SEO-optimized blog posts with AI-generated cover images.

## What This Does

This workflow is basically a robot blogger. Every day at 9 AM, it:

1. **Scrapes news** from RSS feeds and NewsAPI
2. **Filters for relevance** (only fresh articles from the last 48 hours)
3. **Checks your history** via Airtable to avoid duplicate topics
4. **Picks the best SEO topic** using AI (Gemini Pro)
5. **Writes a full article** with Claude 3.5 Sonnet, researching facts via Perplexity
6. **Generates a cover image** using Flux/Fal.ai
7. **Uploads everything to Ghost** as a draft

Then you wake up, review the draft, hit publish. Done.

## Why I Built This

I run a tech blog on Ghost and got tired of the daily grind:
- Manually checking news sources
- Picking topics that haven't been covered
- Writing, researching, formatting
- Finding or creating images
- Uploading to Ghost

The whole process took 2-3 hours per post. This workflow does it in about 5 minutes, completely automated.

The trickiest part was the Ghost image upload (more on that below).

## What You Need

### Required Services & APIs

1. **n8n** (obviously) - Self-hosted or cloud
2. **Ghost Blog** - Your Ghost instance with Admin API access
3. **Airtable** - For tracking post history (free tier works)
4. **NewsAPI** - Get a free API key at [newsapi.org](https://newsapi.org)
5. **OpenRouter** or **OpenAI** - For AI models (Gemini Pro, Claude Sonnet)
6. **Perplexity API** - For research tool
7. **Fal.ai** - For image generation (supports Flux, DALL-E, Midjourney)

### Estimated Costs

- **n8n**: $0 (self-hosted) or ~$20/month (cloud)
- **Ghost**: $0 (self-hosted) or $9-25/month
- **Airtable**: $0 (free tier)
- **NewsAPI**: $0 (free tier, 100 requests/day)
- **OpenRouter**: ~$0.50-2 per article (depends on models)
- **Perplexity**: ~$0.10-0.30 per article
- **Fal.ai**: ~$0.05-0.15 per image

**Total per article: ~$0.65-2.50**

Still way cheaper than hiring a writer.

## Setup Instructions

### 1. Import the Workflow

Download `ghost-automated-blog-system.json` and import it into your n8n instance.

### 2. Configure Credentials

You'll need to add these credentials in n8n:

- **Ghost Admin API** (used in multiple nodes)
- **Airtable Personal Access Token**
- **OpenRouter API Key** (or OpenAI)
- **Perplexity API Key**
- **Fal.ai API Key**

### 3. Set Up Airtable

Create a new base with a table that has at least these fields:
- `title` (Single line text)
- `summary` (Long text)

This tracks what you've already written about to avoid duplicates.

In the workflow, update these nodes with your Airtable IDs:
- `Get Topic History`
- `Save to History`

### 4. Configure News Sources

**RSS Feed Source node:**
- Replace the URL with your preferred RSS feed
- Default is TechCrunch, but use whatever fits your niche

**NewsAPI Source node:**
- Add your NewsAPI key
- Adjust the search query `(Technology OR AI OR Webdev)` to match your topics
- Change language if needed (default is English)

### 5. Set Up Ghost URLs

Find and replace `YOUR-GHOST-BLOG.com` in these nodes:
- `Upload to Ghost`
- `Update Post with Image`

Use your actual Ghost domain (e.g., `blog.example.com` or `example.ghost.io`).

### 6. Add Fal.ai Key

In these HTTP nodes, replace `YOUR_FAL_AI_KEY_HERE`:
- `Submit to Flux/Fal.ai`
- `Check Status`
- `Get Result`

### 7. Test Run

Before enabling the daily schedule:
1. Disable the `Schedule: Daily 09:00` trigger
2. Add a manual trigger temporarily
3. Run the workflow once to verify everything works
4. Check your Ghost drafts
5. Once confirmed, re-enable the schedule trigger

## How It Works (Technical Deep Dive)

### Stage 1: News Aggregation

The workflow starts with two parallel news sources:

**RSS Feed Reader:**
- Pulls from any RSS feed you configure
- Default: TechCrunch
- Can add multiple feeds if needed

**NewsAPI:**
- Fetches articles matching your keywords
- Filters by language and recency
- Returns last 48 hours of content

Both sources merge and get normalized into a common format.

### Stage 2: Filtering & Deduplication

**48-Hour Filter:**
- Removes anything older than 2 days
- Ensures content freshness

**Duplicate Remover:**
- Uses fuzzy matching to detect similar titles
- Removes exact URL duplicates
- Similarity threshold: 80% (configurable in code)

### Stage 3: AI Topic Selection

**Agent 1: Topic Scout (Gemini Pro)**
- Receives all filtered news articles
- Gets your Airtable history of already-covered topics
- Analyzes for SEO potential
- Picks ONE best topic you haven't written about
- Returns structured JSON with title, keywords, description

This is the brain of the operation. The prompt is designed to think like an SEO strategist, focusing on search intent rather than just news headlines.

### Stage 4: Content Generation

**Agent 2: Ghost Writer (Claude Sonnet 4.5)**
- Takes the selected topic
- Uses **Perplexity** as a research tool to find facts, stats, benchmarks
- Writes a full blog post in HTML format
- Includes inline citations with hyperlinks
- Automatically creates the post in Ghost via the Ghost Admin API
- Returns the post ID and metadata

The prompt instructs Claude to act like a "CMS Operator" rather than a chatbot, ensuring it actually executes the Ghost tool instead of just returning text.

### Stage 5: Image Generation & Upload

**Agent 3: Image Prompter (Gemini Flash)**
- Receives the article title
- Generates a minimalist, abstract image prompt
- Style: dark mode, premium, cinematic, no people
- Designed for tech blog aesthetics

**Fal.ai Pipeline:**
1. Submits the prompt to Fal.ai (Flux model)
2. Waits 5 seconds for initial processing
3. Polls the status endpoint until image is ready
4. Downloads the generated image as binary data

**Ghost Upload (The Tricky Part):**

Here's where it gets interesting. The native n8n Ghost node doesn't support direct image URL uploads for feature images. You can't just pass an external URL and expect it to work.

The solution is a 3-step custom upload process:

1. **Download Image** - Convert the Fal.ai URL to binary data in n8n
2. **Upload to Ghost** - POST the binary to `/ghost/api/admin/images/upload/`
   - This returns a local Ghost path like `/content/images/2025/01/image.png`
3. **Update Post** - PUT request to update the draft post with the feature image

This ensures the image is properly hosted on your Ghost instance, not externally linked.

### Stage 6: History Tracking

After the post is created, the workflow saves the topic to Airtable with:
- Article title
- Meta description

This prevents the AI from writing about the same topic twice.

## Customization Ideas

### Change the Writing Style

Edit the prompt in `Agent 2: Ghost Writer`:
- Make it more formal/casual
- Add humor or sarcasm
- Focus on different content types (tutorials, listicles, case studies)

### Use Different AI Models

The workflow uses:
- **Gemini Pro** for topic selection (fast, cheap)
- **Claude Sonnet 4.5** for writing (high quality)
- **Gemini Flash** for image prompts (ultra-fast)

You can swap these out in OpenRouter:
- GPT-4 for better reasoning
- Claude Opus for longer content
- Llama for budget builds

### Multi-Language Support

1. Change NewsAPI language parameter
2. Update AI prompts to specify output language
3. Adjust RSS feeds to your target language

### Different Image Styles

Edit the `Agent 3: Image Prompter` prompt:
- Photorealistic instead of abstract
- Illustrations instead of 3D renders
- Specific color schemes
- Brand-consistent styles

### Add More News Sources

You can extend the workflow with:
- Google News RSS
- Reddit API
- Hacker News API
- Twitter/X API
- Custom web scrapers

Just normalize the data format and merge it in.

## Troubleshooting

### "Ghost API returns 422 Unprocessable Entity"

This usually means:
- Invalid credentials
- Wrong Ghost URL format
- Missing required fields in the post

Check the Ghost Admin API logs for details.

### "Image upload fails"

Common issues:
- Fal.ai key is invalid
- Image generation timed out
- Binary data got corrupted

Try increasing the wait time or adding retry logic.

### "AI keeps writing about the same topic"

Check your Airtable:
- Verify the history is actually being saved
- Confirm the merge node is working
- Look at the AI prompt context

### "Costs are too high"

Optimize by:
- Using cheaper models (Gemini instead of GPT-4)
- Reducing Perplexity calls
- Using lower-res images
- Running less frequently

### "Articles are low quality"

Improve the prompt:
- Add more specific instructions
- Include writing samples
- Add constraints (minimum word count, required sections)
- Use Claude Opus instead of Sonnet

## Advanced: Running Multiple Instances

Want to run different blogs or niches?

1. **Duplicate the workflow**
2. **Change the Airtable base** for each instance
3. **Adjust RSS feeds and keywords** per niche
4. **Point to different Ghost blogs** if needed
5. **Offset the schedule** so they don't run simultaneously

Example schedule:
- Tech blog: 9:00 AM
- Business blog: 10:00 AM
- Lifestyle blog: 11:00 AM

## Performance Notes

Full execution time: **~4-6 minutes** depending on:
- News API response time
- AI model speed
- Image generation time
- Airtable query performance

The longest step is usually Claude writing the article (2-3 minutes).

## Known Limitations

1. **No fact-checking** - The AI might hallucinate despite Perplexity research
2. **SEO is basic** - It optimizes for keywords but doesn't do advanced SEO
3. **Images can be hit or miss** - AI art isn't always perfect
4. **No automatic publishing** - Creates drafts only (for safety)
5. **Requires manual review** - You should still check before publishing

## Future Improvements

Things I might add:
- Automatic internal linking to old posts
- Social media auto-posting
- Email newsletter integration
- A/B testing for titles
- Automatic image alt text generation
- Plagiarism checking
- Multi-post generation per day

## Credits & Resources

Built with:
- [n8n](https://n8n.io) - Workflow automation
- [Ghost](https://ghost.org) - Blog platform
- [Anthropic Claude](https://anthropic.com) - AI writing
- [Google Gemini](https://deepmind.google/technologies/gemini/) - Topic selection
- [Perplexity](https://perplexity.ai) - Research
- [Fal.ai](https://fal.ai) - Image generation
- [Airtable](https://airtable.com) - Database

Inspired by all the people selling "premium" n8n workflows for $99. Here it is for free.

## License

Do whatever you want with this. No attribution needed.

If you break your blog or rack up a huge AI bill, that's on you though.

---

Questions? Found a bug? Improved the workflow? Let me know or submit a PR.
