require 'date'
require 'fileutils'

require "bundler/setup"
require "stringex"

deploy_default = "rsync"

## -- Rsync Deploy config -- ##
# Be sure your public key is listed in your server's ~/.ssh/authorized_keys file
ssh_user       = "dh_rhkyg3@pdx1-shared-a4-08.dreamhost.com"
ssh_port       = "22"
document_root  = "/home/dh_rhkyg3/insightfultroll.com"
rsync_delete   = false
rsync_args     = "--omit-dir-times --no-p"  # Any extra arguments to pass to rsync


## -- Rsync Deploy config -- ##
s3_bucket      = "insightfultroll.com"

# This will be configured for you when you run config_deploy
deploy_branch  = "gh-pages"

## -- Misc Configs -- ##

public_dir      = "_site"    # compiled site directory
posts_dir       = "_posts"    # directory for blog files
new_post_ext    = "md"  # default new post file extension when using the new_post task
new_page_ext    = "md"  # default new page file extension when using the new_page task
server_port     = "4000"      # port for preview server eg. localhost:4000


desc "Begin a new post in #{posts_dir}"
task :new_post, :title do |t, args|
  if args.title
    title = args.title
  else
    title = get_stdin("Enter a title for your post: ")
  end

  filename = "#{posts_dir}/#{Time.now.strftime('%Y-%m-%d')}-#{title.to_url}.#{new_post_ext}"
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end
  puts "Creating new post: #{filename}"
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: post"
    post.puts "title: \"#{title.gsub(/&/,'&amp;')}\""
    post.puts "date: #{Time.now.strftime('%Y-%m-%d %H:%M:%S %z')}"
    post.puts "comments: true"
    post.puts "categories: "
    post.puts "---"
  end
end

desc "Generate jekyll site"
task :generate do
  puts "## Generating Site with Jekyll - #{Time.now.strftime("%A %B %d, %Y %H:%M")}"
  system "jekyll build"
end

desc "Default deploy task"
task :deploy do
  # Check if preview posts exist, which should not be published
  if File.exist?(".preview-mode")
    puts "## Found posts in preview mode, regenerating files ..."
    File.delete(".preview-mode")
    Rake::Task[:generate].execute
  end
  Rake::Task["#{deploy_default}"].execute
end


desc "Generate website and deploy"
task :gen_deploy => [ :generate, :deploy] do
end


desc "Deploy website via rsync"
task :rsync do
  exclude = ""
  if File.exist?('./rsync-exclude')
    exclude = "--exclude-from '#{File.expand_path('./rsync-exclude')}'"
  end
  puts "## Deploying website via Rsync"
  ok_failed system("rsync -avze 'ssh -p #{ssh_port}' #{exclude} #{rsync_args} #{"--delete" unless rsync_delete == false} #{public_dir}/ #{ssh_user}:#{document_root}")
end

def ok_failed(condition)
  if (condition)
    puts "OK"
  else
    puts "FAILED"
  end
end

def get_stdin(message)
  print message
  STDIN.gets.chomp
end

def ask(message, valid_options)
  if valid_options
    answer = get_stdin("#{message} #{valid_options.to_s.gsub(/"/, '').gsub(/, /,'/')} ") while !valid_options.include?(answer)
  else
    answer = get_stdin(message)
  end
  answer
end

def blog_url(user, project, source_dir)
  cname = "#{source_dir}/CNAME"
  url = if File.exists?(cname)
    "http://#{IO.read(cname).strip}"
  else
    "http://#{user.downcase}.github.io"
  end
  url += "/#{project}" unless project == ''
  url
end
