# vi: set ft=ruby :
#

node_data_disk_count = 1
driveletters = ('a'..'z').to_a
disk_size = 10 #GB
cpus = 1
memory = 950


node_count = 3# node-0 is our client

Vagrant.configure(2) do |config|
	puts "Creating #{node_count} nodes."
	puts "Creating #{node_data_disk_count} data disks (#{disk_size}G) eachh."

	require "fileutils"
	f = File.open("dist/hosts.ini", "w")
	f.puts "node-0 ansible_host=192.168.250.10"
	f.puts ""
	f.puts "[gluster_servers]"
	(1..node_count).each do |num|
		f.puts "node-#{num} ansible_host=192.168.250.#{num+10}"
	end
	f.close

	(0..node_count).reverse_each do |num|
		config.vm.define "node-#{num}" do |node|
			vm_ip = "192.168.250.#{num+10}"

			node.vm.box = "centos/8"
			node.vm.synced_folder ".", "/vagrant", disabled: true
			node.vm.network :private_network,
				:ip => vm_ip,
				:libvirt__driver_queues => "#{cpus}"
			node.vm.post_up_message = "VM private ip: #{vm_ip}"
			node.vm.hostname = "node-#{num}"

			node.vm.provider "libvirt" do |lvt|
				lvt.qemu_use_session = false
				lvt.storage_pool_name = "default"
				lvt.memory = "#{memory}"
				if num == 0
					lvt.cpus = 2
				else
					lvt.cpus = "#{cpus}"
				end
				lvt.nested = false
				lvt.cpu_mode = "host-passthrough"
				lvt.volume_cache = "writeback"
				lvt.graphics_type = "none"
				lvt.video_type = "vga"
				lvt.video_vram = 1
				lvt.usb_controller :model => "none" # (requires vagrant-libvirt >= 0.44 which is in Fedora 30 and up?)
				lvt.random :model => 'random'
				lvt.channel :type => 'unix', :target_name => 'org.qemu.guest_agent.0', :target_type => 'virtio'
				#disk_config
				if num != 0
					(1..(node_data_disk_count)).each do |d|
						lvt.storage :file, :size => "#{disk_size}G", :serial => "#{d}"
					end #disk_config
				end

			end #libvirt

			if num == 0
				node.vm.synced_folder "./dist", "/vagrant", type: "rsync", create: true, rsync__args: ["--verbose", "--archive", "--delete", "-z"]
				node.vm.post_up_message << "\nYou can now access the nodes (credential for root is 'foobar')"
			end
		end
	end
end
