guard "jekyll-plus",
  :config => ["_config.yml", "_devconfig.yml"],
  :drafts => true do
  # watch /.*html/
  # watch /.*md/
  watch /.*pdf/
  watch /^_config.yml/
  watch /^_drafts/
  watch /^_includes/
  watch /^_layouts/
  watch /^_posts/
  watch /^_sass/
  watch /^css/
  watch /^images/
end

guard :livereload do
  watch /^public/
end
