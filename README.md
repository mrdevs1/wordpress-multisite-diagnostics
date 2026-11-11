# WordPress Multisite Diagnostics

A practical troubleshooting guide for diagnosing, managing, and maintaining WordPress Multisite networks using WP-CLI, PHP, database inspection, and WordPress network administration tools.

## What Is WordPress Multisite?

WordPress Multisite allows multiple websites to operate from a single WordPress installation.

A multisite network shares the WordPress core files while maintaining separate site content and settings.

Common use cases include:

- Managing multiple business websites
- Educational institution networks
- Franchise websites
- Regional or language-specific websites
- Client website networks
- Multi-brand website platforms

## Common Multisite Problems

- Network administration dashboard is inaccessible.
- Subsites return HTTP 404 errors.
- Domain mapping does not work correctly.
- Network-activated plugins fail on individual sites.
- Users cannot access specific subsites.
- Database tables are missing or inconsistent.
- Subsite URLs redirect incorrectly.
- Network configuration becomes corrupted.

## 1. Verify Multisite Configuration

Inspect the `wp-config.php` file for the multisite configuration constants.

```php
define('MULTISITE', true);
define('SUBDOMAIN_INSTALL', false);

define('DOMAIN_CURRENT_SITE', 'example.com');
define('PATH_CURRENT_SITE', '/');

define('SITE_ID_CURRENT_SITE', 1);
define('BLOG_ID_CURRENT_SITE', 1);
```

Important configuration values:

| Constant | Purpose |
|---|---|
| `MULTISITE` | Enables multisite functionality |
| `SUBDOMAIN_INSTALL` | Determines subdomain or subdirectory installation |
| `DOMAIN_CURRENT_SITE` | Network's primary domain |
| `PATH_CURRENT_SITE` | Network installation path |
| `SITE_ID_CURRENT_SITE` | Network ID |
| `BLOG_ID_CURRENT_SITE` | Main site ID |

The configuration must match the actual network installation.

Do not copy these values blindly into an existing production installation.

## 2. List All Network Sites

Use WP-CLI to inspect registered sites:

```bash
wp site list
```

Display useful site information:

```bash
wp site list --fields=blog_id,url,registered,last_updated
```

This helps identify:

- Existing subsites
- Unexpected site URLs
- Recently registered sites
- Sites that have not been updated recently

## 3. Inspect a Specific Subsite

Retrieve information about a particular site:

```bash
wp site get 2
```

Replace `2` with the actual site ID.

Inspect its URL settings:

```bash
wp option get home --url=https://site.example.com
```

```bash
wp option get siteurl --url=https://site.example.com
```

If the URLs are incorrect, investigate network configuration and domain mapping before changing database values.

## 4. Troubleshoot Subsite 404 Errors

Subsite 404 errors commonly occur because of incorrect rewrite rules or network configuration.

Check permalink settings:

```bash
wp rewrite structure --url=https://site.example.com
```

Flush rewrite rules for the affected site:

```bash
wp rewrite flush --url=https://site.example.com
```

Also investigate:

1. Whether the subsite exists in the network.
2. Whether its domain and path are correct.
3. Whether the server rewrite configuration supports multisite.
4. Whether Apache or Nginx rules are configured correctly.
5. Whether a conflicting redirect is affecting the request.

Rewrite rules differ between subdomain and subdirectory installations.

## 5. Inspect Network-Activated Plugins

List network-activated plugins:

```bash
wp plugin list --network
```

Inspect plugins for an individual site:

```bash
wp plugin list --url=https://site.example.com
```

A plugin can be network-activated or activated on individual subsites.

When troubleshooting plugin issues, verify:

- Network activation status
- Plugin compatibility with multisite
- Required network-level configuration
- Site-specific settings
- PHP errors and plugin dependencies

Avoid deactivating network-wide plugins without understanding their impact on the entire network.

## 6. Troubleshoot User Permissions

List users associated with a specific site:

```bash
wp user list --url=https://site.example.com
```

Inspect a user's details:

```bash
wp user get admin --url=https://site.example.com
```

WordPress Multisite has separate site-level roles and network-level administration capabilities.

Important distinctions:

- Super Admin: Manages the entire network.
- Administrator: Manages an individual site.
- Editor: Manages site content.
- Author: Publishes and manages their own posts.
- Contributor: Creates content without publishing.
- Subscriber: Has basic site access.

A site administrator is not automatically a network Super Admin.

Check network permissions before modifying user capabilities.

## 7. Inspect Multisite Database Tables

WordPress Multisite uses shared network tables and site-specific tables.

Common network-level tables:

```text
wp_blogs
wp_blogmeta
wp_site
wp_sitemeta
wp_signups
wp_users
wp_usermeta
```

Site-specific tables commonly follow this pattern:

```text
wp_posts
wp_postmeta
wp_options

wp_2_posts
wp_2_postmeta
wp_2_options

wp_3_posts
wp_3_postmeta
wp_3_options
```

The actual table prefix may differ.

List database tables:

```bash
wp db tables
```

Inspect site registration records:

```sql
SELECT blog_id, site_id, domain, path, registered
FROM wp_blogs
ORDER BY blog_id;
```

This helps identify incorrect domain or path mappings.

Always use the actual database table prefix.

## 8. Troubleshoot Domain Mapping

Modern WordPress Multisite supports custom domains for individual sites.

Inspect a site's registered domain:

```bash
wp site get 2
```

Check its URL options:

```bash
wp option get home --url=https://example.com
```

```bash
wp option get siteurl --url=https://example.com
```

Investigate:

- DNS configuration
- Domain ownership and verification
- SSL certificate validity
- Web server virtual host configuration
- WordPress site domain and path
- Redirect rules
- CDN and reverse proxy configuration

Do not assume that changing the WordPress site URL alone will resolve domain mapping problems.

## 9. Create a New Subsite Using WP-CLI

Create a new site:

```bash
wp site create \
  --slug=store \
  --title="Online Store"
```

For a subdirectory network, the resulting URL may look like:

```text
https://example.com/store/
```

For a subdomain network, it may look like:

```text
https://store.example.com/
```

The actual result depends on the network configuration.

Verify the new site:

```bash
wp site list
```

## 10. Inspect Network Options

Retrieve a network-level option:

```bash
wp network meta get 1 site_name
```

List network options:

```bash
wp network meta list 1
```

These commands help inspect network metadata.

For individual site settings, use:

```bash
wp option list --url=https://site.example.com
```

Do not confuse network metadata with individual site options.

## 11. Diagnose Multisite Database Issues

Check database integrity:

```bash
wp db check
```

Inspect database size:

```bash
wp db size
```

Check whether expected site tables exist:

```sql
SHOW TABLES LIKE 'wp_2_%';
```

Investigate:

- Missing subsite tables
- Incorrect table prefixes
- Unexpected site registrations
- Orphaned metadata
- Incorrect database permissions
- Database connection failures

Never manually delete multisite database tables without identifying their ownership and dependencies.

## 12. Useful WP-CLI Commands

| Command | Purpose |
|---|---|
| `wp site list` | List network sites |
| `wp site get 2` | Inspect a site |
| `wp site create` | Create a subsite |
| `wp plugin list --network` | List network plugins |
| `wp user list --url=URL` | List site users |
| `wp option get home --url=URL` | Inspect site URL |
| `wp network meta list 1` | Inspect network metadata |
| `wp db check` | Check database integrity |
| `wp db tables` | List database tables |
| `wp rewrite flush --url=URL` | Refresh site rewrite rules |

## Multisite Troubleshooting Checklist

- [ ] Verify multisite configuration constants.
- [ ] Confirm the primary network domain.
- [ ] Check registered subsites.
- [ ] Inspect subsite URL settings.
- [ ] Verify DNS and SSL configuration.
- [ ] Check Apache or Nginx rewrite rules.
- [ ] Inspect network-activated plugins.
- [ ] Verify Super Admin permissions.
- [ ] Review network and site database tables.
- [ ] Check database integrity.
- [ ] Test affected sites individually.
- [ ] Verify functionality after troubleshooting.

## Important Security Guidelines

1. Always back up the complete network database before making structural changes.
2. Protect network administrator accounts.
3. Avoid granting unnecessary Super Admin privileges.
4. Verify plugin compatibility before network activation.
5. Keep WordPress core, plugins, and themes updated.
6. Use HTTPS for all network sites.
7. Avoid manually modifying shared database tables without understanding their relationships.
8. Test configuration changes in a staging environment.

## Conclusion

WordPress Multisite requires a different troubleshooting approach from a standard WordPress installation.

Understanding network configuration, site registration, domain mapping, database structure, user permissions, and WP-CLI commands makes network administration more reliable.

This repository serves as a practical technical reference for WordPress developers and administrators managing multisite netwo
