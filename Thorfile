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
  @@notification_recipients = %(
    micah@botandrose.com
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
      message = ""
      send_email "voyeur@botandrose.com", "BOTandROSE Voyeur", e, '', subject, message
    end
  end
  
  def send_email(from, from_alias, to, to_alias, subject, message)
  	msg = <<END_OF_MESSAGE
From: #{from_alias} <#{from}>
To: #{to_alias} <#{to}>
Subject: #{subject}
	
#{message}
END_OF_MESSAGE
	
	  Net::SMTP.start('staging.botandrose.com') do |smtp|
		  smtp.send_message msg, from, to
	  end
  end

end

class Host
  attr_accessor :name
  
  def initialize(name)
    self.name = name
  end
  
  def down?
   res = Net::HTTP.get_response uri
   not res.code.match /^2../
  end
  
  protected
  
    def uri
      URI.parse "http://#{name}"
    end
end