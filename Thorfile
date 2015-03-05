require 'thor'
require 'NIFTY'
require 'httparty'

class Default < Thor
	desc 'testlb', 'testlb'
	def testlb(url)
		logger.info "testing: #{url}"
		loop {
			begin
				res = HTTParty.get(url)
				logger.info res
			rescue StandardError => e
				logger.error "#{e.class} #{e.message}"
			end
		}
	end

	desc 'bluegreen', 'ニフティクラウドの LB から旧サーバーを切り離し、新サーバーを追加します'
	option :lb_name
	option :lb_port
	option :instance_port
	option :detach_lb_owner
	option :attach_lb_owner
	option :detach, :type => :array
	option :attach, :type => :array
	option :region
	def bluegreen

		@detach_api = get_api(options[:region], ENV['DETACH_ACCESS_KEY_ID'], ENV['DETACH_SECRET_ACCESS_KEY'])
		@attach_api = get_api(options[:region], ENV['ATTACH_ACCESS_KEY_ID'], ENV['ATTACH_SECRET_ACCESS_KEY'])

		logger.info "切替対象サーバーの情報取得中"
		show_instances_info

		start_time_all = Time.now
		start_time_detach = Time.now

		#logger.info "切替処理開始、まずはデタッチ"
		#detach_lb_name = options[:detach_lb_owner] ? "#{options[:detach_lb_owner]}.#{options[:lb_name]}" : options[:lb_name]
		#response = @detach_api.deregister_instances_from_load_balancer(
		#	:load_balancer_name => detach_lb_name,
		#	:instances => options[:detach],
		#	:load_balancer_port => options[:lb_port],
		#	:instance_port => options[:instance_port]
		#)
		#logger.info "デタッチ処理が完了しました (#{Time.now - start_time_detach} 秒)"
		#logger.info response
		#logger.info "次、アタッチ"
		#start_time_attach = Time.now
		#attach_lb_name = options[:attach_lb_owner] ? "#{options[:attach_lb_owner]}.#{options[:lb_name]}" : options[:lb_name]
		#response = @attach_api.register_instances_with_load_balancer(
		#	:load_balancer_name => attach_lb_name,
		#	:instances => options[:attach],
		#	:load_balancer_port => options[:lb_port],
		#	:instance_port => options[:instance_port]
		#)
		#logger.info "アタッチ処理が完了しました (#{Time.now - start_time_attach} 秒)"
		#logger.info response
		#logger.info "切替が完了しました (#{Time.now - start_time_all} 秒)"

		logger.info "次、アタッチ"
		start_time_attach = Time.now
		attach_lb_name = options[:attach_lb_owner] ? "#{options[:attach_lb_owner]}.#{options[:lb_name]}" : options[:lb_name]
		response = @attach_api.register_instances_with_load_balancer(
			:load_balancer_name => attach_lb_name,
			:instances => options[:attach],
			:load_balancer_port => options[:lb_port],
			:instance_port => options[:instance_port]
		)
		logger.info "アタッチ処理が完了しました (#{Time.now - start_time_attach} 秒)"
		logger.info response
		logger.info "切替処理開始、まずはデタッチ"
		detach_lb_name = options[:detach_lb_owner] ? "#{options[:detach_lb_owner]}.#{options[:lb_name]}" : options[:lb_name]
		response = @detach_api.deregister_instances_from_load_balancer(
			:load_balancer_name => detach_lb_name,
			:instances => options[:detach],
			:load_balancer_port => options[:lb_port],
			:instance_port => options[:instance_port]
		)
		logger.info "デタッチ処理が完了しました (#{Time.now - start_time_detach} 秒)"
		logger.info response

		logger.info "切替対象サーバーの情報取得中"
		show_instances_info
		exit
	end

	no_commands do
		def show_instances_info
			detach_api = get_api(options[:region], ENV['DETACH_ACCESS_KEY_ID'], ENV['DETACH_SECRET_ACCESS_KEY'])
			attach_api = get_api(options[:region], ENV['ATTACH_ACCESS_KEY_ID'], ENV['ATTACH_SECRET_ACCESS_KEY'])

			detach_instances = [detach_api.describe_instances.reservationSet.item].flatten.map {|item| item.instancesSet.item }.flatten
			detach_instances = detach_instances.select {|i| options[:detach].include? i.instanceId  }
			detach_instances.each do |i|
				logger.info "デタッチ対象 #{i.instanceId} LB 情報: #{i.loadbalancing.inspect}"
			end
			attach_instances = [attach_api.describe_instances.reservationSet.item].flatten.map {|item| item.instancesSet.item }.flatten
			attach_instances = attach_instances.select {|i| options[:detach].include? i.instanceId  }
			attach_instances.each do |i|
				logger.info "アタッチ対象 #{i.instanceId} LB 情報: #{i.loadbalancing.inspect}"
			end
		end

		def logger
			@logger ||= Logger.new(STDOUT)
		end

		def get_api(region, access_key_id, secret_access_key)
			NIFTY::Cloud::Base.new(
				:server => "#{region}.cp.cloud.nifty.com",
				:path => '/api',
				:access_key => access_key_id,
				:secret_key => secret_access_key,
				:socket_timeout => 60*10,
				:connection_timeout => 60*10
			)
		end
	end
end
