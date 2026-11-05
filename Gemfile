source "https://rubygems.org"

# Jekyll 4.x — built and deployed by GitHub Actions (.github/workflows/pages.yml),
# not by the legacy "github-pages" gem, so we are not pinned to Jekyll 3.x.
gem "jekyll", "~> 4.4"
gem "minima", "~> 2.5"

group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.17"
  gem "jekyll-seo-tag", "~> 2.8"
  gem "jekyll-sitemap", "~> 1.4"
end

# Ruby 3.x no longer bundles webrick, which `jekyll serve` needs.
gem "webrick", "~> 1.8"

# Timezone data on Windows / JRuby.
platforms :windows, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end
