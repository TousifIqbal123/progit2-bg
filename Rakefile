# coding: utf-8
require 'octokit'
require 'github_changelog_generator'

def exec_or_raise(command)
  puts `#{command}`
  if (! $?.success?)
    raise "'#{command}' failed"
  end
end

module GitHubChangelogGenerator

  #OPTIONS = %w[ user project token date_format output
  #              bug_prefix enhancement_prefix issue_prefix
  #              header merge_prefix issues
  #              add_issues_wo_labels add_pr_wo_labels
  #              pulls filter_issues_by_milestone author
  #              unreleased_only unreleased unreleased_label
  #              compare_link include_labels exclude_labels
  #              bug_labels enhancement_labels
  #              between_tags exclude_tags exclude_tags_regex since_tag max_issues
  #              github_site github_endpoint simple_list
  #              future_release release_branch verbose release_url
  #              base configure_sections add_sections]

  def get_log(&task_block)
    options = Parser.default_options
    yield(options) if task_block

    options[:user],options[:project] = ENV['TRAVIS_REPO_SLUG'].split('/')
    options[:token] = ENV['GITHUB_API_TOKEN']
    options[:unreleased] = false

    generator = Generator.new options
    generator.compound_changelog
  end

  module_function :get_log
end

namespace :book do
  desc 'build basic book formats'
  task :build do

    begin
      version_string = ENV['TRAVIS_TAG'] || `git describe --tags`.chomp
      if version_string.empty?
        version_string = '0'
      end
      date_string = Time.now.strftime("%Y-%m-%d")
      params = "--attribute revnumber='#{version_string}' --attribute revdate='#{date_string}'"
      puts "Generating contributors list"
      `git shortlog -s | grep -v -E "(Straub|Chacon)" | cut -f 2- | column -c 120 > book/contributors.txt`

      puts "Converting to HTML..."
      `bundle exec asciidoctor #{params} -a data-uri progit.asc`
      puts " -- HTML output at progit.html"

      puts "Converting to EPub..."
      `bundle exec asciidoctor-epub3 #{params} progit.asc`
      puts " -- Epub output at progit.epub"

      puts "Converting to Mobi (kf8)..."
      `bundle exec asciidoctor-epub3 #{params} -a ebook-format=kf8 progit.asc`
      puts " -- Mobi output at progit.mobi"

      puts "Converting to PDF... (this one takes a while)"
      `bundle exec asciidoctor-pdf #{params} progit.asc 2>/dev/null`
      puts " -- PDF output at progit.pdf"

    end
  end
end



task :default => "book:build"
