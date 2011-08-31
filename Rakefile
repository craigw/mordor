require 'open-uri'
require 'fileutils'
require 'erb'

module Convert
  def to_longitude s
    s =~ /^(\d+)-(\d+)(?:-\d+)?([EW])/ or return 9999.9999
    ($3 == 'E' ? 1.0 : -1.0) * ($1.to_f + $2.to_f / 60.0)
  end

  def to_latitude s
    s =~ /^(\d+)-(\d+)(?:-\d+)?([SN])/ or return 9999.9999
    ($3 == 'N' ? 1.0 : -1.0) * ($1.to_f + $2.to_f / 60.0)
  end

  extend self
end

TEMPLATE = ERB.new <<-ERB
require 'ostruct'

module Mordor
  module Station
    def self.find cccc
      LIST[cccc.to_s.upcase]
    end

    LIST = {}
    <% stations.each do |station_data|
      station_data.strip!
      code, a, b, name, state, country, c, latitude_s, longitude_s = station_data.split(/\;/)
    %>
    LIST["<%= code %>"] = OpenStruct.new :code => "<%= code %>",
                                     :name => "<%= name %>",
                                     :state => "<%= state %>",
                                     :country => "<%= country %>",
                                     :latitude => <%= Convert.to_latitude latitude_s %>,
                                     :longitude => <%= Convert.to_longitude longitude_s %>,
                                     :raw => <%= station_data.inspect %>
    <% end %>
    LIST.freeze
  end
end
ERB

CACHE = './nsd_cccc.txt'

task :clean do
  if FIle.exists? CACHE
    puts "Removing #{CACHE}"
    File.unlink CACHE
  end
end

task :download do
  if !File.exists? CACHE
    puts "Downloading #{CACHE}"
    data = open('http://weather.noaa.gov/data/nsd_cccc.txt').read
    File.open CACHE, 'w' do |f|
      f.puts data
    end
  end
end

task :generate => :download do
  puts "Generating mordor/station.rb"
  data = File.read CACHE
  stations = data.split(/\n/)
  FileUtils.mkdir 'mordor' unless File.exists? 'mordor'
  File.open 'mordor/station.rb', 'w+' do |f|
    f.puts TEMPLATE.result binding
  end
end

task :default => :generate
