require 'yaml'

task :sort do
  puts 'Loading dictionary...'
  dictionary = YAML.load_file 'dictionary.yml'
  sorted_dictionary = {}
  puts 'Sorting...'
  dictionary.sort_by { |key, value| key.downcase }.each do |key, value|
    sorted_dictionary[key] = value
  end
  puts 'Writing dictionary...'
  File.open('dictionary.yml', 'w') { |file| file.write sorted_dictionary.to_yaml }
  puts 'All done!'
end
