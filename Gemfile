source "https://rubygems.org"

gem "jekyll", "~> 4.4"

group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.17"
  gem "jekyll-sitemap"
  gem "jekyll-tidy"
  # htmlbeautifier >= 1.4 (pulled in by jekyll-tidy) fails with "Unmatched
  # sequence" on this site's generated HTML; keep the 1.3 series.
  gem "htmlbeautifier", "~> 1.3.1"
end

# Renders math blocks at build time (_config.yml: math_engine: katex).
# Requires a JavaScript runtime (node) on the build machine.
gem "kramdown-math-katex"

# Windows and JRuby do not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
platforms :windows, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.2", :platforms => [:windows]
