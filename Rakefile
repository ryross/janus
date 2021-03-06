module VIM
  Dirs = %w[ after autoload doc plugin ruby snippets syntax ftdetect ftplugin colors indent ]
end

directory "tmp"
VIM::Dirs.each do |dir|
  directory(dir)
end

def vim_plugin_task(name, repo=nil)
  cwd = File.expand_path("../", __FILE__)
  dir = File.expand_path("tmp/#{name}")
  subdirs = VIM::Dirs

  namespace(name) do
    if repo
      file dir => "tmp" do
        if repo =~ /git$/
          sh "git clone #{repo} #{dir}"

        elsif repo =~ /download_script/
          if filename = `curl --silent --head #{repo} | grep attachment`[/filename=(.+)/,1]
            filename.strip!
            sh "curl #{repo} > tmp/#{filename}"
          else
            raise ArgumentError, 'unable to determine script type'
          end

        elsif repo =~ /(tar|gz|vba|zip)$/
          filename = File.basename(repo)
          sh "curl #{repo} > tmp/#{filename}"

        else
          raise ArgumentError, 'unrecognized source url for plugin'
        end

        case filename
        when /zip$/
          sh "unzip -o tmp/#{filename} -d #{dir}"

        when /tar\.gz$/
          dirname  = File.basename(filename, '.tar.gz')

          sh "tar zxvf tmp/#{filename}"
          sh "mv #{dirname} #{dir}"

        when /vba(\.gz)?$/
          if filename =~ /gz$/
            sh "gunzip -f tmp/#{filename}"
            filename = File.basename(filename, '.gz')
          end

          # TODO: move this into the install task
          mkdir_p dir
          lines = File.readlines("tmp/#{filename}")
          current = lines.shift until current =~ /finish$/ # find finish line

          while current = lines.shift
            # first line is the filename, followed by some unknown data
            file = current[/^(.+?)\s+\[\[\[(\d+)$/, 1]

            # then the size of the payload in lines
            current = lines.shift
            num_lines = current[/^(\d+)$/, 1].to_i

            # the data itself
            data = lines.slice!(0, num_lines).join

            # install the data
            Dir.chdir dir do
              mkdir_p File.dirname(file)
              File.open(file, 'w'){ |f| f.write(data) }
            end
          end
        end
      end

      task :pull => dir do
        if repo =~ /git$/
          Dir.chdir dir do
            sh "git pull"
          end
        end
      end

      task :install => [:pull] + subdirs do
        Dir.chdir dir do
          if File.exists?("Rakefile") and `rake -T` =~ /^rake install/
            sh "rake install"
          elsif File.exists?("install.sh")
            sh "sh install.sh"
          else
            subdirs.each do |subdir|
              if File.exists?(subdir)
                sh "cp -rf #{subdir}/* #{cwd}/#{subdir}/"
              end
            end
          end
        end

        yield if block_given?
      end
    else
      task :install => subdirs do
        yield if block_given?
      end
    end
  end

  desc "Install #{name} plugin"
  task name do
    puts
    puts "*" * 40
    puts "*#{"Installing #{name}".center(38)}*"
    puts "*" * 40
    puts
    Rake::Task["#{name}:install"].invoke
  end
  task :default => name
end

vim_plugin_task "ack.vim",          "http://github.com/mileszs/ack.vim.git"
vim_plugin_task "color-sampler",    "http://www.vim.org/scripts/download_script.php?src_id=12179"
vim_plugin_task "conque",           "http://conque.googlecode.com/files/conque_1.1.tar.gz"
vim_plugin_task "fugitive",         "http://github.com/tpope/vim-fugitive.git"
vim_plugin_task "git",              "http://github.com/tpope/vim-git.git"
vim_plugin_task "haml",             "http://github.com/tpope/vim-haml.git"
vim_plugin_task "indent_object",    "http://github.com/michaeljsmith/vim-indent-object.git"
vim_plugin_task "javascript",       "http://github.com/pangloss/vim-javascript.git"
vim_plugin_task "markdown_preview", "http://github.com/robgleeson/vim-markdown-preview.git"
vim_plugin_task "nerdtree",         "http://github.com/scrooloose/nerdtree.git"
vim_plugin_task "nerdcommenter",    "http://github.com/scrooloose/nerdcommenter.git"
vim_plugin_task "surround",         "http://github.com/tpope/vim-surround.git"
vim_plugin_task "taglist",          "http://vim.sourceforge.net/scripts/download_script.php?src_id=7701"
vim_plugin_task "vividchalk",       "http://github.com/tpope/vim-vividchalk.git"
vim_plugin_task "supertab",         "http://github.com/ervandew/supertab.git"
vim_plugin_task "cucumber",         "http://github.com/tpope/vim-cucumber.git"
vim_plugin_task "textile",          "http://github.com/timcharper/textile.vim.git"
vim_plugin_task "rails",            "http://github.com/tpope/vim-rails.git"
vim_plugin_task "rspec",            "http://github.com/taq/vim-rspec.git"
vim_plugin_task "zoomwin",          "http://www.vim.org/scripts/download_script.php?src_id=9865"
vim_plugin_task "snipmate",         "http://github.com/msanders/snipmate.vim.git"
vim_plugin_task "autoclose",        "http://github.com/Townk/vim-autoclose.git"
vim_plugin_task "markdown",         "http://github.com/tpope/vim-markdown.git"
vim_plugin_task "align",            "http://github.com/tsaleh/vim-align.git"

vim_plugin_task "command_t",        "http://github.com/wincent/Command-T.git" do
  sh "find ruby -name '.gitignore' | xargs rm"
  Dir.chdir "ruby/command-t" do
    if `rvm > /dev/null 2>&1` && $?.exitstatus == 1
      sh "rvm system ruby extconf.rb"
    else
      sh "/usr/bin/ruby extconf.rb" # assume /usr/bin/ruby is system ruby
    end
    sh "make clean && make"
  end
end

vim_plugin_task "janus_themes" do
  # custom version of railscasts theme
  File.open(File.expand_path("../colors/railscasts+.vim", __FILE__), "w") do |file|
    file.puts <<-VIM.gsub(/^ +/, "").gsub("<SP>", " ")
      runtime colors/railscasts.vim
      let g:colors_name = "railscasts+"

      set fillchars=vert:\\<SP>
      set fillchars=stl:\\<SP>
      set fillchars=stlnc:\\<SP>
      hi  StatusLine guibg=#cccccc guifg=#000000
      hi  VertSplit  guibg=#dddddd
    VIM
  end

  # custom version of jellybeans theme
  File.open(File.expand_path("../colors/jellybeans+.vim", __FILE__), "w") do |file|
    file.puts <<-VIM.gsub(/^      /, "")
      runtime colors/jellybeans.vim
      let g:colors_name = "jellybeans+"

      hi  VertSplit    guibg=#888888
      hi  StatusLine   guibg=#cccccc guifg=#000000
      hi  StatusLineNC guibg=#888888 guifg=#000000
    VIM
  end
end

vim_plugin_task "molokai" do
  sh "curl http://www.vim.org/scripts/download_script.php?src_id=9750 > colors/molokai.vim"
end

vim_plugin_task "mustasche" do
  sh "curl http://github.com/defunkt/mustache/raw/master/contrib/mustache.vim > syntax/mustache.vim"
end

vim_plugin_task "ryderbeans" do
  sh "curl https://gist.github.com/raw/660525/451f3d81a4cbc98402d608dac274ae819a2bf6ee/ryderbeans.vim > colors/ryderbeans.vim"
end

desc "Cleanup all the files"
task :clean do
  rm_rf "tmp"
end

desc "Update the documentation"
task :update_docs do
  puts "Updating VIM Documentation..."
  system "vim -e -s <<-EOF\n:helptags ~/.vim/doc\n:quit\nEOF"
end

desc "link vimrc to ~/.vimrc"
task :link_vimrc do
  %w[ vimrc gvimrc ].each do |file|
    dest = File.expand_path("~/.#{file}")
    unless File.exist?(dest)
      ln_s(File.expand_path("../#{file}", __FILE__), dest)
    end
  end
end

task :default => [
  :update_docs,
  :link_vimrc
]
