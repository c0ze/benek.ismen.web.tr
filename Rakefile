require "aws-sdk"
require "yaml"

def traverse_directory(path)
  Dir.entries(path).map do |f|
    next if [".", ".."].include? f
    f_path = File.join(path, f)
    if File.directory? f_path
      traverse_directory f_path
    else
      f_path
    end
  end
end

desc "Deploy via S3"
task :s3 do

  config = YAML.load_file "./_awsconfig.yml"
  local_dir = './_site'

  access_key = config['aws']['access_key']
  secret_key = config['aws']['secret_key']
  region = config['aws']['region']
  bucket_name = config['aws']['bucket_name']

  s3 = Aws::S3::Client.new(
    region: region,
    credentials: Aws::Credentials.new(access_key, secret_key)
  )

  page = s3.list_objects(bucket: bucket_name)

  p "deleting content"
  loop do
    page.contents.each do |ob|
      s3.delete_object bucket: bucket_name, key: ob.key
    end
    break unless page.next_page?
    page = page.next_page
  end

  p "uploading _site"
  traverse_directory(local_dir).flatten.compact.each do |f|
    key = f.gsub(local_dir+"/", "")
    s3.put_object bucket: bucket_name, key: key, body: File.open(f), acl: "public-read"
  end

  p "s3 deploy complete"
end
