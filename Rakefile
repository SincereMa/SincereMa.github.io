require 'rake'
require 'date'

desc "Begin a new post in the _posts directory"
task :post do
  title = ENV.fetch('title', 'new-post')
  date = ENV.fetch('date', DateTime.now.strftime('%Y-%m-%d'))
  filename = "#{date}-#{title}.md"
  post_path = ENV.fetch('posts_dir', '_posts')

  # 确保路径存在
  unless Dir.exist?(post_path)
    abort "rake aborted: '#{post_path}' directory not found."
  end

  # 创建 Markdown 文件头信息
  header = <<~HEREDOC
  ---
  layout: post
  title: "#{title}"
  date: "#{date}"
  categories:
  tags:
  thumbs:
  ---
  
  YOUR CONTENT HERE...
  HEREDOC

  # 生成文件
  File.write(File.join(post_path, filename), header)
  puts "新 post '#{title}' 已经创建到 #{post_path}/#{filename}"
end
