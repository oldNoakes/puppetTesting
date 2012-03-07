require 'rubygems'
require 'puppet'
require 'colored'
require 'rake/clean'

desc "verifies correctness of node syntax"
task :verify_nodes, [:manifest_path, :module_path, :nodename_filter] do |task, args|
  fail "manifest_path must be specified" unless args[:manifest_path]
  fail "module_path must be specified" unless args[:module_path]
 
  @manifest_path = args[:manifest_path]
  @module_path = args[:module_path]
  @nodename_filter = args[:nodename_filter]

  nodes = collect_puppet_nodes
  puts "Found: #{nodes.length} nodes to evaluate".cyan
  verify_successful = true
  collect_puppet_nodes.each do |nodename|
    print "Verifying node #{nodename}: ".cyan
    begin
      compile_catalog(nodename)
      puts "[ok]".green
    rescue => error
      #puts "#{nodename} failed to compile due to: #{error.message}"
      puts "[FAILED] - #{error.message}".red
      verify_successful = false
    end
  end
  raise "[Compilation Failure] - Check output for details" unless verify_successful
end

def compile_catalog(nodename)
  Puppet.settings.handlearg("--config", ".")
  Puppet.parse_config

  Puppet.settings.handlearg("--manifest", @manifest_path)
  Puppet.settings.handlearg("--modulepath", @module_path)

  node = Puppet::Node.new(nodename)
  node.merge 'architecture' => 'x86_64', 'ipaddress' => '127.0.0.1', 'hostname' => nodename, 'fqdn' => "#{nodename}.localdomain", 'operatingsystem' => 'redhat'
  Puppet::Resource::Catalog.indirection.find(node.name, :use_node => node)
end

def collect_puppet_nodes
  Puppet.settings.handlearg("--manifest", @manifest_path)
  parser = Puppet::Parser::Parser.new("environment")
  nodes = parser.environment.known_resource_types.nodes.keys
  nodes.select { |node| node =~ /#{@nodename_filter}/ } unless @nodename_filter.nil?
end

