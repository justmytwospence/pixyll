#!/usr/bin/env rake

desc "Build site with drafts"
task :build do
  require "jekyll"
  Jekyll::Commands::Build.process({})
end

desc "Automate incremental rebuilds with live reloading"
task :watch do
  require "guard"
  require "guard/commander"
  Guard.start({
    "no_interactions" => true
  })
end

desc "Deploy site to github-pages"
task :deploy => :build do
  require "jekyll"
  require "octopress-deploy"
  Octopress::Deploy.push({})
end
