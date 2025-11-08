# WordPress Database Optimization

## Overview

This document covers database optimization strategies and procedures for jochen-schweizer-corporate.de. Regular database maintenance improves site performance, reduces server load, and ensures efficient data storage.

## Table of Contents

1. [Autoloaded Options Cleanup](#autoloaded-options-cleanup)
2. [Transients Cleanup](#transients-cleanup)
3. [Post Revisions Management](#post-revisions-management)
4. [Database Table Optimization](#database-table-optimization)
5. [Orphaned Data Cleanup](#orphaned-data-cleanup)

---

## Autoloaded Options Cleanup

### What Are Autoloaded Options?

WordPress autoloaded options are configuration settings loaded with every page request. Excessive autoloaded options significantly impact performance by increasing memory usage and database query time.

### Investigation Commands

#### 1. Count Current Autoloaded Options

```bash
wp option list --autoload=yes --format=count
```

#### 2. List All Autoloaded Options

```bash
wp option list --autoload=yes --format=table --fields=option_name,size_bytes
```

#### 3. Export to CSV for Analysis

```bash
wp option list --autoload=yes --format=csv --fields=option_name,size_bytes > autoloaded_options_$(date +%Y%m%d).csv
```

#### 4. Find Large Autoloaded Options

```bash
wp option list --autoload=yes --format=table --fields=option_name,size_bytes | sort -k2 -n | tail -20
```

#### 5. Check Active Plugins

```bash
wp plugin list --status=active --format=table
```

#### 6. Check Active Theme

```bash
wp theme list --status=active
```

### Common Old Plugin Patterns to Check

When cleaning up, look for these patterns from inactive plugins and themes:

- `wprc_t_expired_*` - Plugin cache entries
- `woo_*` - WooThemes plugins/themes (NOT WooCommerce)
- `widget_*` - Check against active sidebars first
- `_transient_*`, `_transient_timeout_*` - Expired transients
- `aioseop_*` - All In One SEO Pack (if using different SEO plugin)
- `_yoast_*` - Yoast SEO (if using different SEO plugin)
- `w3tc_*` - W3 Total Cache
- `wp_super_cache_*` - WP Super Cache
- `wordfence_*` - Wordfence (if removed)
- `jetpack_*` - Jetpack (if removed)
- `icl_*`, `wpml_*` - WPML (if removed)
- Theme-specific prefixes from old themes

### Safe Cleanup Process

#### Step 1: Backup Database

**CRITICAL:** Always backup before any cleanup!

```bash
# Export database
wp db export backup_$(date +%Y%m%d_%H%M%S).sql
```

#### Step 2: Identify Options to Remove

Check each pattern against active plugins and themes:

```bash
# Example: Check for specific pattern
wp option list --autoload=yes --format=csv --fields=option_name | grep '^old_plugin_'

# Count matches
wp option list --autoload=yes --format=csv --fields=option_name | grep '^old_plugin_' | wc -l
```

#### Step 3: Remove Inactive Plugin Options

```bash
# Delete options matching a pattern
wp option list --autoload=yes --format=csv --fields=option_name | grep '^old_plugin_' | xargs -I {} wp option delete {} 2>/dev/null

# Or delete specific options
wp option delete option_name1 option_name2 option_name3
```

#### Step 4: Verify Results

```bash
# Count options after cleanup
wp option list --autoload=yes --format=count

# Check specific pattern was removed
wp option list --autoload=yes --format=csv --fields=option_name | grep '^old_plugin_'
```

### Safety Guidelines

**DO NOT DELETE:**
- WordPress core options (siteurl, home, blogname, active_plugins, etc.)
- Options from active plugins
- Options from active theme
- Current widget configurations

**ALWAYS:**
- Backup database first
- Test on staging/development environment if possible
- Verify plugin/theme is truly inactive
- Document what you remove
- Monitor site after cleanup

---

## Transients Cleanup

### What Are Transients?

Transients are temporary cached data that WordPress stores in the database. Expired transients should be automatically deleted but often accumulate over time.

### Investigation Commands

#### Count Transients

```bash
# Count all transients
wp transient list --format=count

# Count expired transients
wp db query "SELECT COUNT(*) FROM wp_options WHERE option_name LIKE '_transient_timeout_%' AND option_value < UNIX_TIMESTAMP();"
```

#### List Transient Sizes

```bash
wp db query "SELECT option_name, LENGTH(option_value) as size_bytes FROM wp_options WHERE option_name LIKE '_transient_%' ORDER BY size_bytes DESC LIMIT 20;"
```

### Cleanup Commands

```bash
# Delete all expired transients
wp transient delete --expired

# Delete all transients (will be regenerated as needed)
wp transient delete --all

# Delete specific transient
wp transient delete transient_name
```

---

## Post Revisions Management

### Investigation Commands

#### Count Total Revisions

```bash
wp db query "SELECT COUNT(*) FROM wp_posts WHERE post_type = 'revision';"
```

#### Count by Post Type

```bash
wp db query "SELECT p.post_type, COUNT(r.ID) as revision_count FROM wp_posts r JOIN wp_posts p ON r.post_parent = p.ID WHERE r.post_type = 'revision' GROUP BY p.post_type;"
```

#### Find Posts with Most Revisions

```bash
wp db query "SELECT post_parent, COUNT(*) as revision_count FROM wp_posts WHERE post_type = 'revision' GROUP BY post_parent ORDER BY revision_count DESC LIMIT 20;"
```

### Cleanup Options

#### Delete Old Revisions (Keep Recent)

```bash
# Using WP-CLI plugin (wp-cli/extension-command)
wp post delete $(wp post list --post_type='revision' --format=ids --post_date_before="6 months ago") --force

# Or via database (keep last 5 revisions per post)
# Requires custom query - use with caution
```

#### Limit Future Revisions

Add to wp-config.php:

```php
// Limit to 5 revisions
define('WP_POST_REVISIONS', 5);

// Or disable revisions entirely (not recommended)
define('WP_POST_REVISIONS', false);
```

---

## Database Table Optimization

### Investigation Commands

#### Check Table Sizes

```bash
wp db query "SELECT table_name AS 'Table', ROUND(((data_length + index_length) / 1024 / 1024), 2) AS 'Size (MB)' FROM information_schema.TABLES WHERE table_schema = DATABASE() ORDER BY (data_length + index_length) DESC;"
```

#### Check Table Overhead

```bash
wp db query "SELECT table_name, data_free FROM information_schema.tables WHERE table_schema = DATABASE() AND data_free > 0;"
```

### Optimization Commands

#### Optimize All Tables

```bash
wp db optimize
```

#### Repair Tables (if needed)

```bash
wp db repair
```

#### Check Database

```bash
wp db check
```

---

## Orphaned Data Cleanup

### Orphaned Post Meta

#### Find Orphaned Post Meta

```bash
wp db query "SELECT COUNT(*) FROM wp_postmeta pm LEFT JOIN wp_posts p ON pm.post_id = p.ID WHERE p.ID IS NULL;"
```

#### Delete Orphaned Post Meta

```bash
wp db query "DELETE pm FROM wp_postmeta pm LEFT JOIN wp_posts p ON pm.post_id = p.ID WHERE p.ID IS NULL;"
```

### Orphaned Term Relationships

#### Find Orphaned Term Relationships

```bash
wp db query "SELECT COUNT(*) FROM wp_term_relationships tr LEFT JOIN wp_posts p ON tr.object_id = p.ID WHERE p.ID IS NULL;"
```

#### Delete Orphaned Term Relationships

```bash
wp db query "DELETE tr FROM wp_term_relationships tr LEFT JOIN wp_posts p ON tr.object_id = p.ID WHERE p.ID IS NULL;"
```

### Orphaned User Meta

#### Find Orphaned User Meta

```bash
wp db query "SELECT COUNT(*) FROM wp_usermeta um LEFT JOIN wp_users u ON um.user_id = u.ID WHERE u.ID IS NULL;"
```

#### Delete Orphaned User Meta

```bash
wp db query "DELETE um FROM wp_usermeta um LEFT JOIN wp_users u ON um.user_id = u.ID WHERE u.ID IS NULL;"
```

### Orphaned Comment Meta

#### Find Orphaned Comment Meta

```bash
wp db query "SELECT COUNT(*) FROM wp_commentmeta cm LEFT JOIN wp_comments c ON cm.comment_id = c.comment_ID WHERE c.comment_ID IS NULL;"
```

#### Delete Orphaned Comment Meta

```bash
wp db query "DELETE cm FROM wp_commentmeta cm LEFT JOIN wp_comments c ON cm.comment_id = c.comment_ID WHERE c.comment_ID IS NULL;"
```

---

## Maintenance Schedule

### Weekly Tasks

- Delete expired transients: `wp transient delete --expired`
- Check for spam comments: `wp comment list --status=spam --format=count`

### Monthly Tasks

- Review autoloaded options count
- Check database table sizes
- Delete spam/trash comments older than 30 days

### Quarterly Tasks

- Full autoloaded options audit
- Review and clean post revisions
- Optimize database tables
- Clean orphaned metadata

### Annual Tasks

- Complete database audit
- Archive old data if needed
- Review and update optimization strategies

---

## Performance Monitoring

### Baseline Metrics

Before optimization, record these metrics:

```bash
# Autoloaded options count
wp option list --autoload=yes --format=count

# Autoloaded options total size
wp db query "SELECT SUM(LENGTH(option_value)) as total_bytes FROM wp_options WHERE autoload = 'yes';"

# Total database size
wp db size --human-readable

# Table count
wp db query "SELECT COUNT(*) FROM information_schema.tables WHERE table_schema = DATABASE();"

# Post revision count
wp db query "SELECT COUNT(*) FROM wp_posts WHERE post_type = 'revision';"
```

### After Optimization

Re-run the same commands and compare results. Document improvements in this file.

---

## Safety Checklist

Before any database cleanup:

- [ ] Full database backup created
- [ ] Commands tested on staging/development (if available)
- [ ] Active plugins and themes identified
- [ ] Specific options/data to remove identified
- [ ] Understanding of what each option does
- [ ] Maintenance mode enabled (for major cleanups)
- [ ] Monitoring ready for post-cleanup verification

After cleanup:

- [ ] Site loads correctly
- [ ] Admin area works properly
- [ ] Frontend displays correctly
- [ ] No PHP errors in logs
- [ ] Forms and contact features work
- [ ] E-commerce functions work (if applicable)

---

## Resources

- [WP-CLI Option Commands](https://developer.wordpress.org/cli/commands/option/)
- [WP-CLI Database Commands](https://developer.wordpress.org/cli/commands/db/)
- [WP-CLI Transient Commands](https://developer.wordpress.org/cli/commands/transient/)
- [WordPress Database Optimization](https://developer.wordpress.org/advanced-administration/performance/optimization/)
- Reference: imagewize.com autoload cleanup (1,273 → 831 options, 34.7% reduction)

---

## Notes

- This site appears to be hosted on Plesk (based on error log paths)
- WordPress installation is in `/var/www/vhosts/jochen-schweizer-corporate.de/httpdocs/`
- Always ensure you have SSH access and proper database credentials before starting
- Consider implementing automated monitoring for autoloaded options growth
