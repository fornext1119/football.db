puts "before require 'active_record'"
require 'active_record'
puts "after require 'active_record'"

puts "before require 'active_support/all'"
require 'active_support/all'
puts "after require 'active_support/all'"

puts "before require 'worlddb'"
require 'worlddb'  
puts "after require 'worlddb'"

puts "before require 'sportdb'"
require 'sportdb'  
puts "after require 'sportdb'"


#######################################
# build script (Ruby make)
#
#  use:
#   $ rake   or
#   $ rake build     - to build football.db from scratch
#
#   $ rake update    - to update football.db
#
#   $ rake book    - build book
#
#   $ rake -T        - show all tasks


BUILD_DIR = "./build"

# -- output db config
FOOTBALL_DB_PATH = "#{BUILD_DIR}/football.db"


require './settings'
require './scripts/stats'

DB_CONFIG = {
  adapter:    'sqlite3',
  database:   FOOTBALL_DB_PATH
}


task :default => :build

directory BUILD_DIR


task :clean do
  rm FOOTBALL_DB_PATH if File.exists?( FOOTBALL_DB_PATH )
end



task :env => BUILD_DIR do
  pp DB_CONFIG
  ActiveRecord::Base.establish_connection( DB_CONFIG )  
end

task :config  => :env  do
  logger = LogUtils::Logger.root
  # logger.level = :info

  ## log all warns, errors, fatals to db
  LogDb.setup   
  logger.warn "Rakefile - #{Time.now}"  # say hello; log to db (warn level min)  
end

task :configworld => :config do
  logger = LogUtils::Logger.root
  logger.level = :info
end

task :configsport => :config do
  logger = LogUtils::Logger.root
  logger.level = :debug
end


task :create => :env do
  LogDb.create
  ConfDb.create
  TagDb.create
  WorldDb.create
  PersonDb.create
  SportDb.create
end


task :importworld => :configworld do
  # populate world tables
  WorldDb.read_setup( 'setups/sport.db.admin', WORLD_DB_INCLUDE_PATH, skip_tags: true )
end


task :importbuiltin => :env do
  SportDb.read_builtin
end

############################################
# add more tasks (keep build script modular)

Dir.glob('./tasks/**/*.rake').each do |r|
  puts "  importing task >#{r}<..."
  import r
  # see blog.smartlogicsolutions.com/2009/05/26/including-external-rake-files-in-your-projects-rakefile-keep-your-rake-tasks-organized/
end


############################
# todo: squads - move to worldcup.rake ??

task :squads => :players do
  ## test/try squads for worldcup 2014
  ## add importbuiltin dep too (or reuse from players) ?? why? why not??

  SportDb.read_setup( 'setups/all',          NATIONAL_TEAMS_INCLUDE_PATH )
  SportDb.read_setup( 'setups/2014_squads',  WORLD_CUP_INCLUDE_PATH )
end



################################
#  football clubs n leagues

## fix: move to tasks/setups

task :clubs => :importbuiltin do
  ## todo/fix: add es,it,at too!!!
  SportDb.read_setup( 'setups/teams', EN_INCLUDE_PATH )
  SportDb.read_setup( 'setups/teams', DE_INCLUDE_PATH )
  SportDb.read_setup( 'setups/teams', MX_INCLUDE_PATH )
  SportDb.read_setup( 'setups/teams', BR_INCLUDE_PATH )
  SportDb.read_setup( 'setups/all',   WORLD_INCLUDE_PATH )
end

task :world => :importbuiltin do
  SportDb.read_setup( 'setups/teams',   WORLD_INCLUDE_PATH )
end


task :europe_clubs => :importbuiltin do
  SportDb.read_setup( 'setups/teams',  WORLD_INCLUDE_PATH )   ## use teams_europe ??
  SportDb.read_setup( 'setups/all',    AT_INCLUDE_PATH )
  SportDb.read_setup( 'setups/all',    DE_INCLUDE_PATH )
  SportDb.read_setup( 'setups/all',    EN_INCLUDE_PATH )
  SportDb.read_setup( 'setups/all',    ES_INCLUDE_PATH )
  SportDb.read_setup( 'setups/all',    IT_INCLUDE_PATH )
end


#########################################################
# note: change deps to what you want to import for now

##
# default to worldcup (if no key given)
#
# e.g. use like
#  $ rake update DATA=admin  or
#  $ rake build  DATA=all
#  etc.


DATA_KEY = ENV['DATA'] || ENV['DATASET'] || ENV['FX'] || ENV['FIXTURES'] || 'worldcup'
puts "  using DATA_KEY >#{DATA_KEY}<"

task :importsport => [:configsport, DATA_KEY.to_sym] do
  # nothing here
end


task :deletesport => :env do
  SportDb.delete!
end


desc 'build football.db from scratch (default)'
task :build => [:clean, :create, :importworld, :importsport] do
  puts 'Done.'
end

desc 'update football.db'
task :update => [:deletesport, :importsport] do
  puts 'Done.'
end


# desc 'pull (auto-update) football.db from upstream sources'
# task :pull => :env do
#  SportDb.update!
#  puts 'Done.'
# end
