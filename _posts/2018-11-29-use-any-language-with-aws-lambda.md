---
layout: post
title:  "You can now use Ruby (and any other language you want) with AWS Lambda via the Runtime API"
date:   2018-11-29
---

<span class="dropcap">A</span>WS re:Invent has had many great announcements so far, such as [Amazon Personalize][amazon-personalize]{:target="_blank"}, Amazon's machine learning recommendation engine, and [Amazon Forecast][amazon-forecast]{:target="_blank"}, a deep learning service for time-series data forecasting.

However, my favorite announcement so far has been that [AWS Lambda will now be able to support any programming language of your choosing][aws-lambda-language]{:target="_blank"}. Additionally, [support for Ruby 2.5 is already available][lambda-ruby]{:target="_blank"}. 

Additional languages for AWS Lambda will be supported via the 'Runtime API'. If you are curious to see what the implementation of the Runtime API looks like, Amazon has published a [C++][c-runtime]{:target="_blank"} and [Rust][rust-runtime]{:target="_blank"} variant of the Runtime API implementation. Support for PHP, Erlang, Elixir, Cobol, and NSolid are in the pipeline, but as a developer, you now have the power to include your own custom runtime if you wish. 

However, there are a few requirements that need to be met to use your own runtime. Your Lambda function must include an executable file called **bootstrap**. This executable is responsible for the communication between your code (in the language of your choice) and the AWS Lambda environment via an HTTP based interface. The other caveat being that whatever you are using to execute your code must be able to run in the Lambda environment (e,g, it has to run in a Linux-based environment). For example, your language's interpreter needs to be able to run in the Lambda environment. If you are curious to see what the implementation of a custom runtime looks like, check out the Github links for C++ and Rust above. 

If that wasn't exciting enough, AWS has also announced another feature for AWS Lambda called "Lambda Layers". This allows for shared code to be extracted into a separate Lambda Layer for re-use without the need to package the code with every deployment. This also means that if you are using a custom runtime, you could store it inside a Lambda Layer instead of repackaging it with each function. For those using Node.js, this could be a great way to share NPM modules across multiple functions. For those of you using the Serverless Framework, you will be excited to know that [Serverless has announced immediate support for Lambda Layers][serverless-lambda-layers]{:target="_blank"}.

For those of you who have used AWS Lambda extensively, at some point you would have probably run into the issue of long deployment times. If you have used Node.js with AWS Lambda, you are probably familiar with how large Node module dependency trees can be. With this new feature, sharing libraries and business logic across multiple functions through a separate layer should greatly speed up deployment times, in addition to enforcing separation of concerns between your business logic and your dependencies.


[amazon-personalize]: https://aws.amazon.com/blogs/aws/amazon-personalize-real-time-personalization-and-recommendation-for-everyone/
[amazon-forecast]: https://aws.amazon.com/blogs/aws/amazon-forecast-time-series-forecasting-made-easy/
[aws-lambda-language]: https://aws.amazon.com/blogs/aws/new-for-aws-lambda-use-any-programming-language-and-share-common-components/
[lambda-ruby]: https://aws.amazon.com/blogs/compute/announcing-ruby-support-for-aws-lambda/
[c-runtime]: https://github.com/awslabs/aws-lambda-cpp
[rust-runtime]: https://github.com/awslabs/aws-lambda-rust-runtime
[serverless-lambda-layers]: https://serverless.com/blog/publish-aws-lambda-layers-serverless-framework/
