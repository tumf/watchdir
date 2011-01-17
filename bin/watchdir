#!/usr/bin/env ruby
# $Id: watchdir,v 1.6 2009/03/01 10:39:36 tumf Exp $
# e.g.1:
#   watchdir -e php,yaml,ini -s 2 -c "symfony sync stage go"
# e.g.2:
#   watchdir -e php -c "php -l $$" -f
# 
require 'pp'
require 'optparse'

sleep_sec = 2
command = false
glob_pattern = "./**/*"
$debug = false
extensions = []
directories = []
directory = "."
$flush = false
$growl = false

opt = OptionParser.new
opt.on('--help', 'show this message') { puts opt; exit }
opt.on('-e EXT1,EXT2...',"--extensions=EXT1,EXT2...",
       "comma separated extention(s)"){ |v|
  v.split(',').each{ |e|    extensions << "*." + e }
}
opt.on('-c COMMAND','--command=COMMAND','execute when updated'){ |v| command = v}
opt.on('-s SEC','--sleep=SEC',"default #{sleep_sec} sec."){|v| sleep_sec = v.to_i}
opt.on('-d DIR1,DIR2...','--dir=DIR1,DIR2...',
       "comma separated directories"){ |v| 
  directories = v.split(',') }

opt.on('-f','--flush'){ |v| $flush = true }
opt.on('--growl'){ |v| $growl = true }

opt.on('--debug','debug mode'){ |v| $debug = true }

opt.parse!(ARGV)
patterns = []
if directories.size > 0
  directories.each do |d|
    patterns <<
      extensions.collect{ |e| [d,'**',e].join('/') }
  end
end
p patterns  if $debug

glob_pattern = patterns.join("\0") if patterns.size > 0
p glob_pattern if $debug

watch_list = Hash.new
watch_list.default = 0
last_watch_list = false
mode = :all
mode = :each if command =~ /\$\$/

def command_exec command
  puts("\033[2J") if $flush
    puts("@" + Time.now.to_s)
  if $growl
    message = `#{command}`
    unless message == @last_message
      system("growlnotify -t '%s' -m '%s'" % [command,message])
    end
    @last_message = message
    puts(message)
  else
    system(command) 
  end
end

begin
  while true 
    watch_list.clear
    Dir.glob(glob_pattern).each do |file| 
      watch_list[file] = File.mtime(file)
    end
    # p watch_list if $debug
    last_watch_list = watch_list.clone unless last_watch_list
    updated = false
    watch_list.each do |file,time|
      if last_watch_list[file] != time
        pp file if $debug
        if mode == :each
          command_exec command.gsub(/\$\$/,file)
        end
        updated = true
      end
    end
    if mode == :all and updated
      if last_watch_list and command
        command_exec command
      end
    end
    last_watch_list = watch_list.clone
    sleep(sleep_sec)
  end
end
