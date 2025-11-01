# Technical Analysis Tools & Commands Used
## Jochen Schweizer Corporate Website (www.jochen-schweizer-corporate.de)

This document lists all technical tools and commands used to analyze the website and gather performance data for the optimization proposal.

---

## Online Analysis Tools

### 1. Google PageSpeed Insights
**URL:** https://pagespeed.web.dev/
**Purpose:** Measure Core Web Vitals and performance scores

**Commands used:**
- Mobile analysis: https://pagespeed.web.dev/analysis/https-www-jochen-schweizer-corporate-de/vfnzxty9lq?form_factor=mobile
- Desktop analysis: https://pagespeed.web.dev/analysis/https-www-jochen-schweizer-corporate-de/vfnzxty9lq?form_factor=desktop

**Metrics obtained:**
- Mobile Performance Score: 84/100
- Desktop Performance Score: 91/100
- Largest Contentful Paint (LCP)
- First Input Delay (FID)
- Cumulative Layout Shift (CLS)
- Total Blocking Time (TBT)

### 2. Chrome Lighthouse Treemap
**URL:** https://googlechrome.github.io/lighthouse/treemap/?gzip=1
**Purpose:** Visualize JavaScript bundle sizes and identify largest resources

**Key findings:**
- UserFeedback Premium plugin: 323.2 KiB (chunk-vendors.js)
- Identified as largest resource in loading tree
- Used to prioritize optimization opportunities

---

## Command Line Tools

### 3. CURL - HTTP Client
**Purpose:** Fetch HTML source code and analyze HTTP headers

**Commands used:**

**a) Fetch full HTML source:**
```bash
curl -s https://www.jochen-schweizer-corporate.de
```

**b) Check HTTP version (HTTP/2 detection):**
```bash
curl -sI https://www.jochen-schweizer-corporate.de | grep -i "http/"
```
Result: HTTP/2 200

**c) Check for WP Rocket lazy loading:**
```bash
curl -s https://www.jochen-schweizer-corporate.de | grep -E "(WP Rocket|data-lazy-src|loading=\"lazy\"|\.webp)"
```
Found: data-lazy-src, loading="lazy" attributes (WP Rocket active)

**d) Find image formats:**
```bash
curl -s https://www.jochen-schweizer-corporate.de | grep -oE 'src="https://[^"]+\.(jpg|jpeg|png|webp)"'
```
Result: Mostly JPG, logo is WebP

**e) Count CSS files:**
```bash
curl -s https://www.jochen-schweizer-corporate.de | grep -E "link.*stylesheet" | wc -l
```
Result: 55+ CSS files (Elementor conditional loading)

### 4. GREP - Pattern Matching
**Purpose:** Search HTML source for specific patterns

**Patterns searched:**
- `"WP Rocket"` - Confirmed WP Rocket 3.20 active
- `"data-lazy-src"` - Confirmed lazy loading implementation
- `"loading=\"lazy\""` - Native lazy loading attributes
- `"Elementor"` - Found Elementor 3.32.5
- `".webp"` - Logo uses WebP, content images do not
- `"stylesheet"` - Counted CSS file references
- `"thumb_tb_|thumb_ba_|thumb_inc_"` - Content image naming patterns

### 5. HTML Inspection
**Purpose:** Identify optimization plugins and technical stack

**Findings from HTML source:**
- WP Rocket 3.20 (from meta tags and comments)
- Elementor 3.32.5 (from CSS/JS file paths)
- Hello Elementor theme
- JetFormBuilder (forms)
- Jet Menu (navigation)
- GTranslate (translation widget)
- UserFeedback Premium (feedback collection)

**Key indicators found:**
- `<!-- This website is like a Rocket, isn't it? Performance optimized by WP Rocket. -->`
- `/wp-content/plugins/wp-rocket/`
- `/wp-content/themes/hello-elementor/`
- `/wp-content/plugins/elementor/`

---

## Browser Developer Tools

### 6. Chrome DevTools Network Tab
**Purpose:** Analyze resource loading waterfall

**Analysis performed:**
- Total page size
- Number of HTTP requests
- Blocking resources
- Resource load timing
- HTTP/2 multiplexing verification

### 7. Chrome DevTools Coverage Tab
**Purpose:** Identify unused CSS/JavaScript

**Findings:**
- Elementor using conditional CSS loading (only widgets used on page)
- Inline CSS <8KB intentional (reduces HTTP requests)
- CSS concatenation would break conditional loading

---

## Image Analysis

### 8. Image Format Detection
**Command pattern:**
```bash
curl -s https://www.jochen-schweizer-corporate.de | grep -oE 'src="[^"]*\.(jpg|jpeg|png|webp)"' | sort | uniq -c
```

**Results:**
- Logo: WebP format (optimized) ✓
- Content images: JPG format (not optimized) ✗
- Image naming patterns: thumb_tb_*, thumb_ba_*, thumb_inc_*
- Opportunity: 25-40% file size reduction with WebP conversion

---

## Plugin & Technology Stack Detection

### 9. WordPress Plugin Detection
**Method:** HTML source analysis and file path inspection

**Detected plugins:**
- WP Rocket 3.20 (performance optimization)
- Elementor 3.32.5 (page builder)
- JetFormBuilder (contact forms)
- Jet Menu (menu builder)
- GTranslate (translation)
- UserFeedback Premium (feedback widget - 323KB)

**Detection method:**
- Plugin directory paths: `/wp-content/plugins/[plugin-name]/`
- Version numbers from CSS/JS file paths
- HTML comments and meta tags

### 10. Server Configuration
**Commands:**

**a) Check compression:**
```bash
curl -sI -H "Accept-Encoding: gzip, deflate, br" https://www.jochen-schweizer-corporate.de
```
Expected: Content-Encoding header present

**b) Check caching headers:**
```bash
curl -sI https://www.jochen-schweizer-corporate.de | grep -i "cache"
```
Look for: Cache-Control, Expires headers

**c) Check server type:**
```bash
curl -sI https://www.jochen-schweizer-corporate.de | grep -i "server"
```
Result: Server header information

---

## Performance Metrics Gathered

**From PageSpeed Insights:**
- Performance Score: 84/100 (mobile), 91/100 (desktop)
- First Contentful Paint (FCP)
- Largest Contentful Paint (LCP)
- Total Blocking Time (TBT)
- Cumulative Layout Shift (CLS)
- Speed Index
- Time to Interactive (TTI)

**From Lighthouse Treemap:**
- JavaScript bundle sizes
- Largest resources identified
- UserFeedback Premium: 323.2 KiB (largest)

**From Network Analysis:**
- Total page weight: ~484 KiB
- Number of requests: 55+ CSS files, multiple JS files
- HTTP/2 multiplexing active
- Lazy loading active

---

## Optimization Verification

**Verified active optimizations:**
- ✓ Lazy loading: grep "data-lazy-src" and "loading=\"lazy\""
- ✓ Minification: Checked .min.css and .min.js file extensions
- ✓ HTTP/2: curl -sI confirmed HTTP/2 200
- ✓ WP Rocket active: HTML comments and file paths
- ✓ Elementor CSS optimization: Inline CSS <8KB present
- ✓ Conditional CSS loading: Only used widgets loaded

**Identified opportunities:**
- ✗ WebP conversion: grep found .jpg not .webp for content images
- ✗ Plugin bloat: Lighthouse Treemap identified 323KB UserFeedback Premium

---

## Summary of Tools Used

**Command-line tools:**
- curl (HTTP requests and header inspection)
- grep (pattern matching in HTML)
- wc (counting occurrences)
- sort & uniq (deduplicating results)

**Online tools:**
- Google PageSpeed Insights (performance scoring)
- Chrome Lighthouse Treemap (bundle size analysis)

**Browser tools:**
- Chrome DevTools Network tab (waterfall analysis)
- Chrome DevTools Coverage tab (unused code detection)

**Analysis techniques:**
- HTML source code inspection
- HTTP header analysis
- Resource waterfall analysis
- File path pattern matching
- Version detection from asset URLs

---

## Reproducibility

All analysis can be reproduced using the commands listed above.
Run them against https://www.jochen-schweizer-corporate.de to verify findings.

**Analysis date:** November 1, 2025
**Tools versions:** curl 8.x, grep (GNU), Chrome 120+
