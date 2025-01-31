source 'https://rubygems.org'
before_deploy:
  - yes | gem update --system --force
  - gem install bundler
  - gem install uri
  - gem install logger
group :jekyll_plugins do
#	gem 'wdm', '>= 0.1.0'
    gem 'classifier-reborn'
    gem 'jekyll'
    gem 'jekyll-archives'
    gem 'jekyll-diagrams'
    gem 'jekyll-email-protect'
    gem 'jekyll-feed'
    gem 'jekyll-imagemagick'
    gem 'jekyll-link-attributes'
    gem 'jekyll-minifier'
    gem 'jekyll-paginate-v2'
    gem 'jekyll-scholar'
    gem 'jekyll-sitemap'
    gem 'jekyll-twitter-plugin'
    gem 'jemoji'
    gem 'mini_racer'
    gem 'unicode_utils'
    gem 'webrick'
end
group :other_plugins do
    gem 'feedjira'
    gem 'httparty'
end
