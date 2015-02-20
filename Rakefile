require 'rake'
require 'sequel'
require 'nokogiri'
require 'logger'
require 'pry'

task :default => [:build]

desc 'Build the docset'
task :build => [:clean, :download, :make_database, :build_docset]

task :clean do
  system('rm -rf hts-* backblue.gif fade.gif index.html cookies.txt')
  system('rm -f docSet.dsidx')
  system('rm -rf terraform.io')
end

task :download do
  puts 'Downloading documentation...'
  system('httrack http://terraform.io/docs/ --mirror')
end

task :make_database do
  db = Sequel.connect('sqlite://docSet.dsidx', loggers: [Logger.new(STDOUT)])
  db.run('CREATE TABLE searchIndex(id INTEGER PRIMARY KEY, name TEXT, type TEXT, path TEXT);')
  db.run('CREATE UNIQUE INDEX anchor ON searchIndex (name, type, path);')

  Dir.chdir('terraform.io/docs/') do
    generate_entries(db, path: 'commands/*.html',         type: 'Command',     title_sub: /\w+: /,  skip_file: 'index')
    generate_entries(db, path: 'providers/*/index.html',  type: 'Provider',    title_sub: ' Provider')
    generate_entries(db, path: 'provisioners/*.html',     type: 'Provisioner', title_sub: / Provisioner|Provisioner /, skip_file: 'index')
  end

  # Add guides
  db[:searchIndex] << { name: 'Commands (CLI)', type: 'Guide', path: 'commands/index.html' }
  db[:searchIndex] << { name: 'Configuration',  type: 'Guide', path: 'configuration/index.html' }
  db[:searchIndex] << { name: 'Internals',      type: 'Guide', path: 'internals/index.html' }
  db[:searchIndex] << { name: 'Modules',        type: 'Guide', path: 'modules/index.html' }
  db[:searchIndex] << { name: 'Plugins',        type: 'Guide', path: 'plugins/index.html' }
  db[:searchIndex] << { name: 'Providers',      type: 'Guide', path: 'providers/index.html' }
  db[:searchIndex] << { name: 'Provisioners',   type: 'Guide', path: 'provisioners/index.html' }

  # Fudge bad entries
  db[:searchIndex].where(name: 'Provider', type: 'Provider').update(name: 'Mailgun')

  db.disconnect
end

task :build_docset do
  contents_dir  = 'Terraform.docset/Contents'
  resources_dir = "#{contents_dir}/Resources"
  documents_dir = "#{resources_dir}/Documents"

  system("mkdir -p #{documents_dir}")
  system("cp -r terraform.io/docs/* #{documents_dir}/")
  system("cp Info.plist #{contents_dir}/")
  system("cp docSet.dsidx #{resources_dir}/")
  system('cp icon.png Terraform.docset/')
end

def generate_entries(db, path:, type:, title_sub:, skip_file: nil)
  Dir.glob(path).each do |filename|
    next if skip_file && filename.include?(skip_file)

    doc  = Nokogiri::HTML(File.open(filename))
    name = doc.css('h1').first.content.sub(title_sub, '')

    entry = { name: name, type: type, path: filename }
    puts entry
    db[:searchIndex] << entry
  end
end

def fudge_db(db)
end
