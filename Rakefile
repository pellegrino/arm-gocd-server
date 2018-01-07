require 'erb'

def get_var(name)
  if ENV[name].to_s.strip.empty?
    raise "environment #{name} not specified!"
  else
    ENV[name]
  end
end

gocd_version = get_var('GOCD_VERSION')
download_url = get_var('GOCD_SERVER_DOWNLOAD_URL')
gocd_full_version = get_var('GOCD_FULL_VERSION')
gocd_git_sha = get_var('GOCD_GIT_SHA')

tag = ENV['TAG']

task :create_dockerfile do
  template = File.read('Dockerfile.erb')
  renderer = ERB.new(template, nil, '-')
  File.open('Dockerfile', 'w') do |f|
    f.puts(renderer.result(binding))
  end
end

task :create_readme do
  template = File.read('README.md.erb')
  renderer = ERB.new(template, nil, '-')
  File.open('README.md', 'w') do |f|
    f.puts(renderer.result(binding))
  end
end

task :build_docker_image do
  tag_name = tag || "v#{gocd_version}"
  sh("docker build . -t gocd-server:#{tag_name}")
end

task :commit_dockerfile do
  sh("git add Dockerfile README.md")
  sh("git commit -m 'Update Dockerfile and README.md with GoCD Version #{gocd_version}' --author 'GoCD CI User <godev+gocd-ci-user@thoughtworks.com>'")
end

task :create_tag do
  sh("git tag 'v#{gocd_version}'")
end

task :git_push do
  maybe_credentials = "#{ENV['GIT_USER']}:#{ENV['GIT_PASSWORD']}@" if ENV['GIT_USER'] && ENV['GIT_PASSWORD']
  repo_url = "https://#{maybe_credentials}github.com/#{ENV['REPO_OWNER'] || 'gocd'}/docker-gocd-server"
  sh(%Q{git remote add upstream "#{repo_url}"})
  sh("git push upstream master")
  sh("git push upstream --tags")
end

task :docker_push_experimental => :build_image do
  org = ENV['EXP_ORG'] || 'gocdexperimental'
  sh("docker tag gocd-server:v#{gocd_version} #{org}/gocd-server:v#{gocd_full_version}")
  sh("docker push #{org}/gocd-server:v#{gocd_full_version}")
end

task :docker_push_stable do
  org = ENV['ORG'] || 'gocd'
  experimental_org = ENV['EXP_ORG'] || 'gocdexperimental'
  sh("docker pull #{experimental_org}/gocd-server:v#{gocd_full_version}")
  sh("docker tag #{experimental_org}/gocd-server:v#{gocd_full_version} #{org}/gocd-server:v#{gocd_version}")
  sh("docker push #{org}/gocd-server:v#{gocd_version}")
end

desc "Publish to dockerhub"
task :publish => [:create_dockerfile, :create_readme, :commit_dockerfile, :create_tag, :git_push]

desc "Build an image locally"
task :build_image => [:create_dockerfile, :build_docker_image]
