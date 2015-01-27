require "git"
require "github_api"


desc "Makes a clone of the repositories from github"
task :clone do
  rp = RepoProcessor.new
  rp.get_repos.each do |repo|
    rp.process(repo)
  end

end

class RepoProcessor
  attr_accessor :repos

  def initialize
    @cache_dir = "cache"
    Dir.mkdir @cache_dir unless Dir.exist?(@cache_dir)
  end

  def get_repos
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

  def process(repo)
    if Dir.exist?("#{@cache_dir}/#{repo.name}")
      pull(repo)
    else
      clone(repo)
    end
  end

end
