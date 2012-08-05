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
  @nodename_filter = args[:nodename_filter] || ".*"

  nodes = collect_puppet_nodes
  failed_nodes = {}
  puts "Found: #{nodes.length} nodes to evaluate".cyan
  nodes.each do |nodename|
    print "Verifying node #{nodename}: ".cyan
    begin
      compile_catalog(nodename)
      puts "[ok]".green
    rescue => error
      puts "[FAILED] - #{error.message}".red
      failed_nodes[nodename] = error.message
    end
  end
  puts "The following nodes failed to compile => #{print_hash failed_nodes}".red unless failed_nodes.empty?
  raise "[Compilation Failure] at least one node failed to compile" unless failed_nodes.empty?
end

def print_hash nodes
  nodes.inject("\n") { |printed_hash, (key,value)| printed_hash << "\t #{key} => #{value} \n" }
end

def compile_catalog(nodename)
  Puppet.settings.handlearg("--config", ".")
  Puppet.parse_config

  Puppet.settings.handlearg("--manifest", @manifest_path)
  Puppet.settings.handlearg("--modulepath", @module_path)

  node = Puppet::Node.new(nodename)
  node.merge('architecture' => 'x86_64',
             'ipaddress' => '127.0.0.1',
             'hostname' => nodename,
             'fqdn' => "#{nodename}.localdomain",
             'operatingsystem' => 'redhat',
             'disable_asserts' => 'true')
  Puppet::Resource::Catalog.indirection.find(node.name, :use_node => node)
end

def collect_puppet_nodes
  Puppet.settings.handlearg("--manifest", @manifest_path)
  parser = Puppet::Parser::Parser.new("environment")
  nodes = parser.environment.known_resource_types.nodes.keys
  nodes.select { |node| node =~ /#{@nodename_filter}/ } unless @nodename_filter.nil?
end

