
desc 'Build for deployment (but do not deploy)'
task :build, [:deployment_configuration] => :clean do |t, args|
  args.with_defaults(:deployment_configuration => 'deploy')
  config_file = "_config_#{args[:deployment_configuration]}.yml"

  if rake_running then
    puts "\n\nWarning! An instance of rake seems to be running (it might not be *this* Rakefile, however).\n"
    puts "Building while running other tasks (e.g., preview), might create a website with broken links.\n\n"
    puts "Are you sure you want to continue? [Y|n]"

    ans = STDIN.gets.chomp
    exit if ans != 'Y' 
  end

  jekyll("build --config _config.yml,#{config_file}")
end

desc 'Clean up generated site'
task :clean do
  cleanup
end

# remove generated site
def cleanup
  sh 'rm -rf _site'
end

desc 'Build and deploy to remote server'
task :deploy, [:deployment_configuration] => :build do |t, args|
  args.with_defaults(:deployment_configuration => 'deploy')
  config_file = "_config_#{args[:deployment_configuration]}.yml"

  text = File.read("_config_#{args[:deployment_configuration]}.yml")
  matchdata = text.match(/^deploy_dir: (.*)$/)
  keydata = text.match(/^deploy_key: (.*)$/)
  if matchdata

    if git_requires_attention("master") then
      puts "\n\nWarning! It seems that the local repository is not in sync with the remote.\n"
      puts "This could be ok if the local version is more recent than the remote repository.\n"
      puts "Deploying before committing might cause a regression of the website (at this or the next deploy).\n\n"
      puts "Are you sure you want to continue? [Y|n]"

      ans = STDIN.gets.chomp
      exit if ans != 'Y' 
    end

    deploy_dir = matchdata[1]
    deploy_key = keydata[1]
    sh "rsync -va _site/ ../cafeauchain-backend/public/blog/" # send to local Rails
    sh "rsync -rvz --delete -e 'ssh -i #{deploy_key}' _site/ #{deploy_dir}"
    time = Time.new
    File.open("_last_deploy.txt", 'w') {|f| f.write(time) }
  else
    puts "Error! deploy_url not found in _config_deploy.yml"
    exit 1
  end
end

desc 'Create a post'
task :create_post, [:date, :title, :category, :content] do |t, args|
  if args.title == nil then
    puts "Error! title is empty"
    puts "Usage: create_post[date,title,category,content]"
    puts "DATE and CATEGORY are optional"
    puts "DATE is in the form: YYYY-MM-DD; use nil or empty for today's date"
    puts "CATEGORY is a string; nil or empty for no category"
    exit 1
  end
  if (args.date != nil and args.date != "nil" and args.date != "" and args.date.match(/[0-9]+-[0-9]+-[0-9]+/) == nil) then
    puts "Error: date not understood"
    puts "Usage: create_post[date,title,category,content]"
    puts "DATE and CATEGORY are optional"
    puts "DATE is in the form: YYYY-MM-DD; use nil or the empty string for today's date"
    puts "CATEGORY is a string; nil or empty for no category"
    puts ""

    title = args.title || "title"

    puts "Examples: create_post[\"\",\"#{args.title}\"]"
    puts "          create_post[nil,\"#{args.title}\"]"
    puts "          create_post[,\"#{args.title}\"]"
    puts "          create_post[#{Time.new.strftime("%Y-%m-%d")},\"#{args.title}\"]"
    exit 1
  end

  post_title = args.title.downcase.strip.gsub(' ', '-').gsub(/[^\w-]/, '')
  puts post_title
  post_date = Time.now.strftime("%Y-%m-%d %H:%M:%S %Z")
  puts post_date

  # the destination directory is <<category>>/$post_dir, if category is non-nil
  # and the directory exists; $post_dir otherwise (a category tag is added in
  # the post body, in this case)
  post_category = args.category
  if post_category and Dir.exists?(File.join(post_category, '_posts/')) then
    post_dir = File.join(post_category, '_posts/')
    yaml_cat = nil
  else
    post_dir = '_posts/'
    yaml_cat = post_category ? "category: #{post_category}\n" : nil
  end

  filename = post_date[0..9] + "-" + post_title.downcase.strip.gsub(' ', '-').gsub(/[^\w-]/, '') + '.md'
  puts filename

  # the condition is not really necessary anymore (since the previous
  # loop ensures the file does not exist)
  if not File.exists?(post_dir + filename) then
    File.open(post_dir + filename, 'w') do |f|
      f.puts "---"
      f.puts "title: \"#{post_title}\""
      f.puts "layout: default"
      f.puts yaml_cat if yaml_cat != nil
      f.puts "date: #{post_date}"
      f.puts "---"
      f.puts args.content if args.content != nil
    end  

    puts "Post created under \"#{post_dir}#{filename}\""
  else
    puts "A post with the same name already exists. Aborted."
  end
  # puts "You might want to: edit #{$post_dir}#{filename}"
end

# launch jekyll
def jekyll(directives = '')
  sh 'jekyll ' + directives
end

# check if there is another rake task running (in addition to this one!)
def rake_running
  `ps | grep 'rake' | grep -v 'grep' | wc -l`.to_i > 1
end

def git_local_diffs
  %x{git diff --name-only} != ""
end

def git_remote_diffs branch
  %x{git fetch}
  %x{git rev-parse #{branch}} != %x{git rev-parse origin/#{branch}}
end

def git_repo?
  %x{git status} != ""
end

def git_requires_attention branch
  $git_check and git_repo? and git_remote_diffs(branch)
end
