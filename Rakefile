require "git"
require "github_api"
require 'rsolr'
require 'nokogiri'
require 'dotenv'


Dotenv.load


SOLR_URL = ENV.fetch('SOLR_URL', "http://localhost:8983/solr")
CACHE_DIR = ENV.fetch('CACHE_DIR', "cache")


desc "Makes a clone of the repositories from github"
task :clone do
  rp = Processor.new
  rp.do_it_all

end

# This module clones the various university repositories from the GeoMetadata
# github account.
#
module GitCloner
  attr_accessor :repos

  def get_repos
    puts "Returning array of repo names"
    @repos = Github.repos.list(user: "OpenGeoMetadata")
      .select { |n| n.name.start_with? 'edu.' }
    @repos
  end

  def clone(repo)
    puts "Cloning #{repo.name}"
    Git.clone(repo.clone_url, repo.name, :path => @cache_dir)
  end

  def pull(repo)
    puts "Pulling #{repo.name}"
    r = Git.open("#{@cache_dir}/#{repo.name}")
    o = r.remote(:origin)
    o.fetch
    o.merge "master"
  end

  def update_repo(repo)
    if Dir.exist?("#{@cache_dir}/#{repo.name}")
      puts "pulling #{repo.name}"
      pull(repo)
    else
      puts "cloning #{repo.name}"
      clone(repo)
    end
  end

end


# This module is used to update the solr database.
#
module SolrUpdate
  attr_accessor :files

  def get_files
    puts "Get list of geoblacklight.xml files"
    @files = Dir.glob("#{@cache_dir}/**/geoblacklight.xml")
    @files
  end

  # update the database with the assigned repo
  def solr_update(fn)
    puts "Processing #{fn}"
    if fn =~ /.xml$/
      doc = Nokogiri::XML(File.open(fn, 'rb').read)
      puts "updating #{fn}"
      # @solr.update :data => doc.to_xml    
    elsif fn =~ /.json$/
      doc = JSON.parse(File.open(fn, 'rb').read)
      @solr.add doc
    else
      raise RuntimeError, "Unknown file type: #{fn}"
    end
  end

  # commit the update
  def commit
    puts "Commit the solr update"
    # @solr.commit
  end

  def update_all
    get_files.each.with_index { |file, i| 
      # solr_update(file)
      if i % 1000 == 0
        #commit
        puts "1000 files reached, calling commit"
      end
    }
    #commit 
    puts "Calling commit for final time."
  end
end


module CleanUp
  def clean_solr
    puts "purge solr index"
    # @solr.delete_by_query('*:*')
    # @solr.commit
  end

  def clean_cache_dir
  end
end


# Main processing controller class
#
class Processor
  include GitCloner
  include SolrUpdate

  def initialize
    @cache_dir = CACHE_DIR
    puts "Create the cache dir if needed."
    Dir.mkdir @cache_dir unless Dir.exist?(@cache_dir)
    @solr_url = SOLR_URL
    puts "Connect to Solr @ #{@solr_url}"
    # @solr = RSolr.connect :url => @solr_url
  end

  def do_it_all
    get_repos.each do |repo|
      update_repo(repo)
    end
    update_all
  end

end
