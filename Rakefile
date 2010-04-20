require 'rubygems'
require 'colorize'
require 'erb'
require 'yaml'

BASE_DIR          = Dir.getwd

# StatSVN 
LOGFILE           = "logfile.log"
SVNLOG_COMMAND    = "svn log -v --xml > #{LOGFILE}"
SVNSTATS_COMMAND  = "java -Xmx512m -Xmn512m -XX:+UseParallelGC -jar #{BASE_DIR}/statsvn.jar -verbose "
SVN_DIRNAME       = 'svn_files'
SVNSTAT_CONFIG    = "#{BASE_DIR}/config/svnstat.conf"

CONFIG_FILES      = FileList['config/projects.yml']
OUTPUT_FILE       = 'index.html'
TEMPLATE_FILE     = 'template.html.erb'

task :default => :all

task :all => [:generate_report]

# load the projects.yml file
projects_def = YAML.load_file("#{CONFIG_FILES}")
projects = projects_def['projects']

desc 'Remove all the created directories'
task :clean do
  puts "Cleaning up project directories".colorize(:red)
  projects.each{|project, repo|
    puts "Deleting #{project} files".colorize(:yellow)
    FileUtils.rm_rf("#{BASE_DIR}/#{project}")
  }
  if File.exist? "#{BASE_DIR}/#{OUTPUT_FILE}"
    puts "Cleaning up generated report".colorize(:red)
    FileUtils.rm("#{BASE_DIR}/#{OUTPUT_FILE}")
  end
end

desc 'Clean up the subversion directories (for compressing reports)'
task :clean_svn do
  puts "Cleaning up SVN repo directories".colorize(:red)
  
  projects.each{|project, repo|
    puts "Deleting #{BASE_DIR}/#{project}/svn_files".colorize(:green)
    FileUtils.rm_rf("#{BASE_DIR}/#{project}/svn_files")
  }
  
end

desc 'Create a package of the reports'
task :package => :clean_svn do
  puts "Packaging report".colorize(:red)
  system ("tar -cvpzf statsvn_report.tar.gz --exclude=statsvn_report.tar.gz --exclude=svnstat_readme.txt --exclude=config --exclude=Rakefile --exclude=statsvn.jar ./")
  puts "Finished packaging and compressing #{BASE_DIR}/statsvn_report.tar.gz".colorize(:green)
end


task :build_subdirs do
  puts "Building directory structure for reports".colorize(:red)
  
  # create a directory for the report and an svn checkout
  projects.each {|project, repo| 
    # set a report directory
    report_dir = "#{BASE_DIR}/#{project}"

    if File.directory? report_dir
      puts "#{report_dir} already exists".colorize(:yellow)
    else 
        puts "Creating project report directory structure for #{report_dir}".colorize(:green)
        Dir.mkdir "#{report_dir}"
        Dir.mkdir "#{report_dir}/#{SVN_DIRNAME}"
    end
  }
end

desc 'Update project SVN repos'
task :update_svn => :build_subdirs do
  puts "Retrieving the latest files from the SVN repositories".colorize(:red)
 
  projects.each{|project, repo|
    project_svn = "#{BASE_DIR}/#{project}/#{SVN_DIRNAME}"
    Dir.chdir "#{project_svn}" # go there
    
    # check if a checkout or update needs to occur
    if File.directory? "#{project_svn}/.svn"
      puts "#{project} exists, updating code".colorize(:green)
      system("svn up")
    else
      puts "#{project} not checked out, doing a full checkout".colorize(:blue)
      system("svn co #{repo} .")
    end
  }
end

desc 'Create svn log files for the projects'
task :generate_logfile =>  [:build_subdirs, :update_svn] do 
  puts "Generate log files for projects".colorize(:red)
  
  projects.each{|project, repo|
     
     puts "Generating log for #{project} project".colorize(:green)
     project_svn = "#{BASE_DIR}/#{project}/#{SVN_DIRNAME}"
     Dir.chdir "#{project_svn}" # go there
     
     # generate an svn log in each project directory
     system(SVNLOG_COMMAND)
  }
end

desc 'Build the reports with statsvn'
task :build_report => [:build_subdirs, :update_svn, :generate_logfile] do 
  puts "Creating SVNStats reports".colorize(:red)
  
  projects.each{|project, repo|
    # move into the project dir
    project_dir = "#{BASE_DIR}/#{project}"
    
    # move into the project directory
    Dir.chdir project_dir
    
    project_svn = "#{project_dir}/#{SVN_DIRNAME}"
    
    puts "Generating SVNStats report for #{project}".colorize(:green)
    system "#{SVNSTATS_COMMAND} -title #{project} -config-file #{SVNSTAT_CONFIG} #{project_svn}/#{LOGFILE} #{project_svn}"
  }
  
end

desc 'Generate navigation page from ERB template'
task :generate_report => [:build_subdirs, :update_svn, :generate_logfile, :build_report] do 
  timestamp =  Time.now.strftime("%a, %d %b %Y, %l:%M%P").squeeze(' ')
  
  template = ERB.new(File.read("#{BASE_DIR}/#{TEMPLATE_FILE}"))
  
  File.open("#{BASE_DIR}/#{OUTPUT_FILE}", 'w'){ |f| f.write(template.result(binding))}
  puts "Finished report page. \n\tYou can access the project reports by opeing in #{Dir.getwd}/index.html".colorize(:green)
end

desc 'Generate navigation page from ERB template'
task :generate_nav_only do 
  
  timestamp =  Time.now.strftime("%a, %d %b %Y, %l:%M%P").squeeze(' ')
  
  template = ERB.new(File.read("#{BASE_DIR}/#{TEMPLATE_FILE}"))
  
  File.open("#{BASE_DIR}/#{OUTPUT_FILE}", 'w'){ |f| f.write(template.result(binding))}
  puts "Finished report page. \n\tYou can access the project reports by opeing in #{Dir.getwd}/index.html".colorize(:green)
end

desc 'List the projects currently defined'
task :list_projects do
   
  puts "Currently defined projects".colorize(:red)
  
  max_len = 10
  # get the longest project name
  projects.each do |k,v|
    if k.length > max_len 
      max_len = k.length
    end
  end

  projects.each do |project, repo|
    puts sprintf "\t%#{max_len}s\t in %s", project, repo
  end
end
