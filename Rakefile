#!/usr/bin/env rake

desc "Build site with drafts"
task :build do
  require "jekyll"
  Jekyll::Commands::Build.process({"show_drafts" => true})
end

desc "Automate incremental rebuilds with live reloading"
task :watch do
  require "guard"
  require "guard/commander"
  Guard.start({"no_interactions" => true})
end

desc "Deploy site to github-pages"
task :deploy do
  require "jekyll"
  require "octopress"
  Jekyll::Commands::Build.process({})
  Octopress::Deploy::push({
    "git_branch" => "gh-pages",
    "git_url" => "git@github.com:justmytwospence/pixyll.git",
    "method" => "git",
    "site_dir" => "public"
  })
end
