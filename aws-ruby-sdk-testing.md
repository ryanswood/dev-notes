AWS Ruby SDK Stubbing

Ways to stub:

Standard stubbing
1. ```
s3 = Aws::S3::Client.new(stub_responses: true)

bucket_data = s3.stub_data(:list_buckets, :buckets => [{name:'aws-sdk'}, {name:'aws-sdk2'}])
s3.stub_responses(:list_buckets, bucket_data)
bucket_names = s3.list_buckets.buckets.map(&:name)

# List each bucket by name
bucket_names.each do |name|
  puts name
end
```

Stubbing + inspecting API calls
```
s3 = Aws::S3::Client.new(stub_responses: true)
s3.create_bucket(bucket: "foo")
s3.api_requests.size # => 1
create_bucket_request = s3.api_requests.first
create_bucket_request[:params][:bucket] # => "foo"
```

Stubbing error responses:
```
s3 = Aws::S3::Client.new(stub_responses: true)
s3.stub_responses(:head_bucket, Timeout::Error)

begin
  s3.head_bucket({bucket: 'aws-sdk'})
rescue Exception => ex
  puts "Caught #{ex.class} error calling 'head_bucket' on 'aws-sdk'"
end
```

Dynamic responses and changing test state for validation
```
client.stub_responses(
      :create_bucket, ->(context) {
        name = context.params[:bucket]
        if fake_s3[name]
          'BucketAlreadyExists' # standalone strings are treated as exceptions
        else
          fake_s3[name] = {}
          {}
        end
      }
    )
```

Finding the request that matches
```
create_table_request = stub_client.api_requests.find do |req|
  req[:operation_name] == :create_table
end
```

2. ```

```

Resources:
- https://docs.aws.amazon.com/sdk-for-ruby/v3/developer-guide/stubbing.html
- https://aws.amazon.com/blogs/developer/advanced-client-stubbing-in-the-aws-sdk-for-ruby-version-3/
