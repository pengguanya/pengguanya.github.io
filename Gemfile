# frozen_string_literal: true

source "https://rubygems.org"

gem "jekyll-theme-chirpy", "~> 7.1"

gem "html-proofer", "~> 5.0", group: :test

# Ruby 3.4+ compatibility: these gems are no longer bundled by default
gem "csv"
gem "base64"

platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]

# Note: jekyll-feed is not needed - Chirpy theme provides its own feed.xml template

