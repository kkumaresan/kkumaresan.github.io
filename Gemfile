source "https://rubygems.org"

# Local development only. GitHub Pages ignores this Gemfile and builds the site
# with its own pinned gem set (Jekyll 3.10).
#
# We deliberately do NOT use the `github-pages` gem here: it pins Liquid 4.0.3,
# which calls String#tainted? — removed in Ruby 3.2 — so it cannot run on a
# modern Ruby at all. Jekyll 4 is used locally instead. Nothing in this site's
# templates is Jekyll-4-specific, so both builds produce the same output.
gem "jekyll", "~> 4.4"
gem "webrick", "~> 1.8"

group :jekyll_plugins do
  gem "jekyll-feed"
  gem "jekyll-seo-tag"
  gem "jekyll-sitemap"
end

# Extracted from the standard library in Ruby 3.4+.
gem "csv"
gem "base64"
gem "bigdecimal"
gem "logger"
gem "ostruct"
