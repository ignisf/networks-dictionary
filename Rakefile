# -*- coding: utf-8 -*-
require 'yaml'

task :clean do
  system 'git clean -df'
end

task :check do
  puts 'Parsing dictionary...'
  YAML.parse_file 'dictionary.yml'
  puts 'Done.'
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
