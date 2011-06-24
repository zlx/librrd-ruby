require 'rubygems'
require 'rake'
require 'date'

#############################################################################
#
# Helper functions
#
#############################################################################

def name
  @name ||= Dir['*.gemspec'].first.split('.').first
end

def name_path
  name.gsub('-', '/')
end

def version
  File.read('VERSION').chomp
end

def date
  Date.today.to_s
end

def rubyforge_project
  name
end

def gemspec_file
  "#{name}.gemspec"
end

def gem_file
  "#{name}-#{version}.gem"
end

def replace_header(head, header_name)
  head.sub!(/(\.#{header_name}\s*= ').*'/) { "#{$1}#{send(header_name)}'"}
end

#############################################################################
#
# Standard tasks
#
#############################################################################

task :default => :test

require 'rake/testtask'
task :test => :build_rrd
Rake::TestTask.new(:test) do |test|
  test.libs << '.' << 'lib' << 'test'
  test.pattern = 'test/**/test_*.rb'
  test.verbose = false
end

desc "Built the RRD.so"
task :build_rrd do
  sh "ruby ext/librrd/extconf.rb && make"
end

#############################################################################
#
# Custom tasks (add your own tasks here)
#
#############################################################################

desc "Cleanup compiled stuff"
task :clean do
  sh 'rm -f *.so *.o Makefile mkmf.log'
  sh 'cd ext/librrd/ && rm -f *.so *.o Makefile mkmf.log'
end


#############################################################################
#
# Packaging tasks
#
#############################################################################

desc "Update gemspec, build gem, commit with 'Release x.x.x', create tag, push to github and gemcutter"
task :release => :build do
  unless `git branch` =~ /^\* master$/
    puts "You must be on the master branch to release!"
    exit!
  end
  sh "git commit --allow-empty -a -m 'Release #{version}'"
  sh "git tag v#{version}"
  sh "git push origin master"
  sh "git push origin tag v#{version}"
  sh "gem push pkg/#{name}-#{version}.gem"
end

desc "Update gemspec and build gem"
task :build => :gemspec do
  sh "mkdir -p pkg"
  sh "gem build #{gemspec_file}"
  sh "mv #{gem_file} pkg"
end

desc "Update gemspec with the latest version and file list"
task :gemspec do
  # read spec file and split out manifest section
  spec = File.read(gemspec_file)
  head, manifest, tail = spec.split("  # = MANIFEST =\n")

  # replace name version and date
  replace_header(head, :name)
  replace_header(head, :version)
  replace_header(head, :date)
  #comment this out if your rubyforge_project has a different name
  replace_header(head, :rubyforge_project)

  # determine file list from git ls-files
  files = `git ls-files`.
    split("\n").
    sort.
    reject { |file| file =~ /^\./ }.
    reject { |file| file =~ /^(rdoc|pkg)/ }.
    map { |file| "    #{file}" }.
    join("\n")

  # piece file back together and write
  manifest = "  s.files = %w[\n#{files}\n  ]\n"
  spec = [head, manifest, tail].join("  # = MANIFEST =\n")
  File.open(gemspec_file, 'w') { |io| io.write(spec) }
  puts "Updated #{gemspec_file}"
end
