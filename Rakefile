require 'html-proofer'

task :htmlproofer do
  options = { :only_4xx => true, :allow_hash_href => true }
  HTMLProofer.check_directory("./_site", options).run
end
