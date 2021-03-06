require 'rubygems'

namespace :build do
  task :compile do
    sh 'bundle exec middleman build'
  end

  task html: [:compile] do
    require 'nokogiri'
    require 'mimemagic'

    file = File.read('build/index.html')

    document = Nokogiri::HTML(file)

    # Check for the correct number of pages
    number_of_pages = document.css('.page').count
    if (number_of_pages % 4) > 0
      puts "Number of pages must be a multiple of 4 for printing, but is #{number_of_pages}!"
      exit 1
    end

    # Embed stylesheets
    document.css('link[rel=stylesheet]').each do |link|
      style = Nokogiri::XML::Node.new('style', document)
      style.content = File.read("build/#{link['href']}")
      link.replace(style)
    end

    # Embed images
    document.css('img').each do |image|
      path = "build/#{image['src']}"

      content = File.read(path)
      content_type = MimeMagic.by_path(path)

      data = [content].flatten.pack('m').gsub("\n","")

      image['src'] = "data:#{content_type};base64,#{data}"
    end

    File.open('build/index.html', 'w') do |file|
      file.write(document.to_html)
    end

    puts "Successfully generated the HTML!"
  end

  task pdf: [:html] do
    require 'doc_raptor'

    DocRaptor.api_key(ENV['DOCRAPTOR_API_KEY'])

    options = {
      document_content: File.read('build/index.html'),
      name: 'workbook.pdf',
      document_type: 'pdf',
      test: ENV['TRAVIS_TAG'] == '',
      prince_options: {
      }
    }

    DocRaptor.create(options) do |pdf, response|
      if response.success?
        filename = (ENV['TRAVIS_BRANCH'] && ENV['TRAVIS_BRANCH'] != 'master') ? "workbook-#{ENV['TRAVIS_COMMIT']}.pdf" : 'workbook.pdf'
        File.open("build/#{filename}", 'w+b') { |file| file.write(pdf.read) }

        puts "Successfully generated the PDF (#{filename})!"

        if ENV['TRAVIS']
          puts "After this build finished, the PDF will be available at"
          puts "https://s3.eu-central-1.amazonaws.com/railsgirls-workbook/#{filename}"
        end

        exit 0
      else
        puts "Failed to generate PDF: "
        response['errors']['error'].each do |error|
          puts " * #{error}"
        end
        exit 1
      end
    end
  end
end

task :preview do
  sh 'bundle exec middleman server'
end

task :setup do
  sh 'gem install bundler'
  sh 'bundle'
end

task default: 'build:pdf'