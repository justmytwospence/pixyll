#!/usr/bin/env rake

desc "Build site with drafts"
task :build do
  require "jekyll"
  Jekyll::Commands::Build.process({
    "show_drafts" => true
  })
end

desc "Automate incremental rebuilds with live reloading"
task :watch do
  require "guard"
  require "guard/commander"

  guardfile = <<-EOF
    guard "jekyll-plus", drafts: true do
      watch /^_drafts/
        watch /^_includes/
        watch /^_layouts/
        watch /^_posts/
        watch /^_sass/
    end
    guard :livereload do
      watch /^public/
    end
  EOF

  Guard.start({
    "guardfile_contents" => guardfile,
    "no_interactions" => true
  })

end

desc "Deploy site to github-pages"
task :deploy do
  require "jekyll"
  require "octopress"

  Jekyll::Commands::Build.process({})
  Octopress::Deploy.push({})

end
