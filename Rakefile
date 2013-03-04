# -*- coding: utf-8 -*-
require 'yaml'

desc 'Clean the working directory'
task :clean do
  system 'git clean -df'
end

desc 'Parse the dictionary and check for errors'
task :check do
  puts 'Parsing dictionary...'
  YAML.parse_file 'dictionary.yml'
  puts 'Done.'
end

desc 'Count the entries in the dictionary'
task :count do
  puts "Number of entries: #{load_dictionary.count}"
end

desc 'List entries with missing field. If `all\' is specified, checks for empty entires'
task :with_missing, :field do |task, args|
  args.with_defaults field: 'all'

  dictionary = load_dictionary
  entries = dictionary.select do |key, value|
    if args[:field] == 'all'
      value.nil?
    else
      value.nil? or value[args[:field]].nil?
    end
  end

  print "Entries with missing field `#{args[:field]}': "
  puts entries.keys.join ', '
end

desc 'List entries with translations without accent'
task :without_accent do
  dictionary = load_dictionary
  entries = dictionary.reject do |key, value|
    next true if value.nil?

    case value['превод']
    when NilClass
      true
    when String
      value['превод'].include? "\u0300"
    when Array
      value['превод'].all? { |translation| translation.include? "\u0300" }
    end
  end

  print 'Entries without accented translation: '
  puts entries.keys.join ', '
end

desc 'Get a list of broken references'
task :check_links do
  broken = Hash.new { |hash, key| hash[key] = [] }

  dictionary = load_dictionary
  dictionary.each do |key, value|
    unless value.nil?
      case value['препратка']
      when String
        link = value['препратка']
        broken[link] |= [key] unless dictionary.has_key? link
      when Array
        value['препратка'].each { |link| broken[link] |= [key] unless dictionary.has_key? link }
      end
    end
  end

  broken.each do |key, value|
    puts "Missing `#{key}' is reffered to by:"
    value.each { |term| puts "  #{term}" }
  end
end

desc 'Sort the dictionary'
task :sort do
  dictionary = load_dictionary
  sorted_dictionary = {}

  puts 'Sorting...'
  dictionary.sort_by { |key, value| key.downcase }.each do |key, value|
    sorted_dictionary[key] = value
  end

  store_dictionary sorted_dictionary
end

def load_dictionary(file = 'dictionary.yml')
  print 'Loading dictionary... '
  dictionary = YAML.load_file file
  puts 'Done.'
  dictionary
end

def store_dictionary(dictionary, file = 'dictionary.yml')
  print "Writing dictionary in #{file}... "
  File.open(file, 'w') { |f| f.write dictionary.to_yaml }
  puts 'Done.'
end
