require 'yaml'
require 'net/http'
require 'net/https'
require 'net/smtp'
require 'uri'

class Voyeur < Thor
  config = YAML.load_file 'config.yml'

  @@hostnames = config["victims"]
  @@notification_recipients = config["notification_recipients"]
  
  desc "peep", "peep at all hosts"
  method_options :verbose => :boolean
  def peep
    exit unless online?
    @@hostnames.each do |h|
      host = Host.new h
      puts "peeping #{h}..." if options.verbose?
      if host.down?
        puts "  DOWN! sending notification." if options.verbose?
        notify host
      else
        puts "  success." if options.verbose?
      end
    end
  end
  
  protected
  
  def notify(host)
    @@notification_recipients.each do |e|
      subject = "#{host.name} DOWN!"
      message = host.error
      send_email "voyeur@botandrose.com", "BOTandROSE Voyeur", e, '', subject, message
    end
  end
  
  def send_email(from, from_alias, to, to_alias, subject, message)
    msg = <<MAIL
Subject: #{subject}
From: #{from_alias} <#{from}>
to: #{to_alias} <#{to}>
Date: #{Time.now}

#{message}
MAIL
    Net::SMTP.start "localhost", 25, "localhost.localdomain" do |smtp|
      smtp.send_message msg, from, to
    end
  end

  def online?
    `ping -c1 google.com` =~ /1 packets transmitted, 1 received/
  end
end

class Host
  attr_accessor :name, :error
  
  def initialize(name)
    self.name = name
  end
  
  def down?
    begin
      res = fetch name
    rescue Exception => e
      self.error = e
      return true
    else
      not res.code.match /^2../
    end
  end
  
  protected
  
    def fetch(uri_str, limit = 10)
      # You should choose better exception.
      raise ArgumentError, 'HTTP redirect too deep' if limit == 0

      response = Net::HTTP.get_response(URI.parse(uri_str))
      case response
      when Net::HTTPSuccess     then response
      when Net::HTTPRedirection then fetch(response['location'], limit - 1)
      else
        response.error!
      end
    end

end
