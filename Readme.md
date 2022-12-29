# Steps to attach `lambda function` with `s3-bucket`

1. Sign-in to aws, using credentials on [click here](https://console.aws.amazon.com/).

1. Click on `Services` from navbar.

1. In `All services` find `Storage` on left hand side. Click on Storage and then on right hand side, click on `s3`.

## How to create a `s3-bucket`

1. Create a bucket, on clicking `Create bucket` button. After clicking, provide required information for the bucket.

    * `Bucket name` e.g. `myBucket`    // this will create the bucket with this name, and this name should be unique. You have to provide any name which is not already present in the bucket.
    * `AWS Region` e.g. `AWS Region`   // remember the region you have selected, because in same region you have to create lambda function.
    * `Block all public access`    // uncheck this field and further uncheck 
    * Inside `Block all public access`, check `I acknowledge that the current settings might result in this bucket and the objects within becoming public.`
    * Finally, click on `Create bucket`.
    * Click on bucket you just created.
    * Click on permissions tab.
    * Slide down and find `Object Ownership` . Click on edit.
    * Select `ACLs enabled` , check `I acknowledge that ACLs will be restored`
    * Click on `Save changes`.

## How to create `permissions` to execute lambda function with restricting objects for CRUD operations

1. Click on `Services` from navbar again.

1. In `All services` find `Security, Identity, & Compliance` on left hand side. Click on Compute and then on right hand side, click on `IAM`.

1. On left side-menu , inside `Access management`,  click on `Roles`.

1. Click on `Create role`.
    
    * `Trusted entity type` should be AWS service
    * `Use case` should be `Lambda` 
    * Click on `next` button.
    * If your permission policy isn't created already, you have to create your own `Permissions policies` to attach with the role. if it is already created, skip further steps and go to `How to create a lambda function` section.
    * Click on `Create Policy` button.
    * Select `JSON` instead of `Visual editor` // this simply means we will upload below custom json file . Just replace `<bucket-name>` with `bucket you created above`.
        * e.g. If my bucket name is filmchi-site, it would look like:
            "Resource": "arn:aws:s3:::filmchi-site/original/*"
            ```
            {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Allow",
                        "Action": [
                            "logs:PutLogEvents",
                            "logs:CreateLogGroup",
                            "logs:CreateLogStream"
                        ],
                        "Resource": "arn:aws:logs:*:*:*"
                    },
                    {
                        "Effect": "Allow",
                        "Action": [
                            "s3:GetObject"
                        ],
                        "Resource": "arn:aws:s3:::<bucket-name>/original/*"
                    },
                    {
                        "Effect": "Allow",
                        "Action": [
                            "s3:PutObject",
                            "s3:PutObjectAcl"
                        ],
                        "Resource": "arn:aws:s3:::<bucket-name>/h-upload/*"
                    }
                ]
            }
            
            ```

    * After JSON is fixed,  Click on `Next: Tags`. We can leave it as it is for now. and Click on `Next: Review`.
    * In the `Name` field, Give policy name related to the task. e.g. `AWS-S3-CloudWatch-Policy`.
    * In the `Summary` field, There should be `CloudWatch Logs` and `S3` in `Service` column.
    * Finally after reviewing, click on `Create policy`.
    * If everything goes right until here, you can see your Policy name in `Policies`.
    * Select the policy name you just created, and click on `Next` button.
    * Give any role name with respect to your bucked. e.g. `filmchi-lambda-role` 
    * Click on `Create role` button.
    * Now skip to `How to create a lambda function` section.


## How to create a `lambda function`

1. Click on `Services` from navbar again.

1. In `All services` find `Compute` on left hand side. Click on Compute and then on right hand side, click on `Lambda`.

1. If there is no function with name `compressImageWithPython`, see next line, if there already exist, skip to `How to trigger function`.

1. click on `Create function` button.

    * `Choose one of the following options to create your function.`, select `Author from scratch` which is also selected by-default.
    * Check region on the navbar , this should be same as region you selected for bucket. If it is not same, then click on dropdown and select the particular region.
    * `Function name` e.g. `my-lambda-function`   // provide any name related to the task you want to do.
    * `Runtime`  e.g. `Python 3.7`  //  select Python 3.7, from the dropdown.
    * `Architecture` e.g. `x86_64`
    * `Change default execution role`
        * expand `Change default execution role` , click on `Use an existing role`.
        * `Existing role` , select a role which you created earlier in `How to create permissions to execute lambda function with restricting objects for CRUD operations`, basically role is certain permissions you attach with the lambda function, such as restricting any folder to GET or PUT or DELETE objects from certain buckets.
    * Inside `Code source` section, click on `Upload from` dropdown and select `.zip file`. Upload zip file, which consists compress image function in python.
    *  Below `Code source` section, go to `Runtime settings`, click on `Edit` button.
    * Inside `Change Handler` section, replace `lambda_function.lambda_handler` with `CreateThumbnail.lambda_handler`.

## How to trigger function
   * Now, click on `Configuration` tab.
   * Now, inside `Configuration`, Edit `General configuration` by clicking on `edit` button.
   * Go to `Timeout` and set `2 min 0 sec` as its value in our case `10 min 0 sec` and click on `Save` button.
   * Below `General configuration` tab on left hand side, click on `Triggers`.
   * Then click on `Add trigger`
        * Inside `Trigger configuration`, select `s3` 
        * In `Bucket` section, select bucket from which you want to trigger this function.
        * In `Prefix - optional`, write `hocal/`
        * check `I acknowledge that using ...` 
        * Click on `Add` button

    
## How to test event trigger

1. Go inside your bucket.
1. Create a folder with name as `original`
1. Go inside folder `original`
1. Click on `Upload` button.
1. Click on `Add files` and select one image.
1. Click on `Upload` button.
1. Now, go back to bucket dashboard and wait for a minute.
1. There will be now two folders in bucket... `original` and `h-upload`.
