require "rubygems"
require "bundler/setup"
require "stringex"

source_dir     = 'source'    # source file directory
deploy_dir     = '_deploy'   # deploy directory (for Github pages deployment)
public_dir     = '_site'    # compiled site directory
deploy_default = 'push'
deploy_branch  = 'master'

desc 'Default deploy task'
task :deploy do
  # Check if preview posts exist, which should not be published
  if File.exists?('.preview-mode')
    puts '## Found posts in preview mode, regenerating files ...'
    File.delete('.preview-mode')
    Rake::Task[:generate].execute
  end

  Rake::Task[:copydot].invoke(source_dir, public_dir)
  Rake::Task["#{deploy_default}"].execute
end

desc 'copy dot files for deployment'
task :copydot, :source, :dest do |t, args|
  excluded_files = %w{**/., **/.., **/.DS_Store, **/._*}
  FileList["#{args.source}/**/.*"].exclude(excluded_files).each do |file|
    unless File.directory?(file)
      cp_r file, file.gsub(/#{args.source}/, "#{args.dest}")
    end
  end
end

desc 'Generate jekyll site'
task :generate do
  puts '## Generating Site with Jekyll'
  system 'jekyll build'
end

desc 'deploy public directory to github pages'
multitask :push do
  puts '## Deploying branch to Github Pages '
  puts '## Pulling any updates from Github Pages '
#  cd "#{deploy_dir}" do
#    Bundler.with_clean_env { system 'git pull' }
#  end
  (Dir["#{deploy_dir}/*"]).each { |f| rm_rf(f) }
  Rake::Task[:copydot].invoke(public_dir, deploy_dir)
  puts "\n## Copying #{public_dir} to #{deploy_dir}"
  cp_r "#{public_dir}/.", deploy_dir
  cd "#{deploy_dir}" do
    system 'git add -A'
    message = "Site updated at #{Time.now.utc}"
    puts "\n## Committing: #{message}"
    system "git commit -m \"#{message}\""
    puts "\n## Pushing generated #{deploy_dir} website"
    system "git push origin #{deploy_branch}"
    puts "\n## Github Pages deploy complete"
  end
end
