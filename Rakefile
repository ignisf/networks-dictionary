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
  puts entries.any? ? entries.keys.join(', ') : 'none'
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
  puts entries.any? ? entries.keys.join(', ') : 'none'
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
  store_dictionary sort_dictionary load_dictionary
end

desc 'Convert the dictionary to JSON'
task :json do
  require 'json'
  dictionary = load_dictionary

  groups = Hash.new { |hash, key| hash[key] = {} }

  dictionary.each do |key, value|
    group = key[0..1].downcase.gsub /[^a-zA-Z0-9]/, '_'
    groups[group][key] = value
  end

  groups.each do |group, entries|
    print  "Creating `#{group}.json'... "
    File.open("#{group}.json", 'w') { |file| file.write entries.to_json }
    puts 'Done.'
  end
end

desc 'List dictionary contributors'
task :contributors do
  system 'git shortlog -s -n'
end

desc 'List the bibliography references used in the dictionary'
task :bibliography do
  dictionary = load_dictionary
  puts 'Extracting references...'
  references = [] 
  #references = dictionary.map { |key, value| value['справка'] unless value.nil? or value['справка'].nil? }
  dictionary.each do |key, value|
    unless value.nil?
	  case value['справка']
	  when String
			  ref = value['справка']
			  references << ref unless ref.nil?
	  when Array
			  value['справка'].each { |ref| references << ref unless ref.nil? }
	  end
	end
  end
  puts 'References:'
  references.uniq.compact.each do |reference|
    puts '  ' + reference
  end
end

desc 'Search for an entry'
task :search, :term do |task, args|
  dictionary = load_dictionary
  term = dictionary[args[:term]]

  unless term.nil?
    puts args[:term]
    term.each do |key, value|
      puts "  #{key}: #{value}"
    end
  else
    puts 'Term not found.'
  end
end

desc 'Add an untranslated term to the dictionary'
task :add, :term do |task, args|
  raise 'Term not specified' unless args[:term]
  dictionary = load_dictionary
  raise 'Term already exists' if dictionary.has_key? args[:term]
  dictionary[args[:term]] = nil

  store_dictionary sort_dictionary dictionary
end

def sort_dictionary(dictionary)
  sorted_dictionary = {}

  puts 'Sorting...'
  dictionary.sort_by { |key, value| key.downcase }.each do |key, value|
    sorted_dictionary[key] = value
  end

  sorted_dictionary
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
