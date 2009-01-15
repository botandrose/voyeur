require 'net/http'
require 'net/smtp'
require 'uri'

class Voyeur < Thor
  @@hostnames = %W(
    domaineselections.com
    botandrose.com
    portlandwinegear.com
    graingerstudios.com
  )
  @@notification_recipients = %W(
    originofstorms@gmail.com
  )
  
  desc "peep", "peep at all hosts"
  def peep
    @@hostnames.each do |h|
      host = Host.new h
      puts "peeping #{h}..."
      if host.down?
        puts "  DOWN! sending notification."
        notify host
      else
        puts "  success."
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

end

class Host
  attr_accessor :name, :error
  
  def initialize(name)
    self.name = name
  end
  
  def down?
    begin
      res = Net::HTTP.get_response uri
      not res.code.match /^2../
    rescue Exception => e
      self.error = e
      return true
    end
  end
  
  protected
  
    def uri
      URI.parse "http://#{name}"
    end
end
