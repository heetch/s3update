# s3update

__Enable your Golang applications to self update with S3.__

This package enables our internal tools to be updated when new commits to their master branch are pushed to Github.

Latest binaries are hosted on S3 under a specific bucket along its current version. When ran locally, the binary will
fetch the version and if its local version is older than the remote one, the new binary will get fetched and will exit,
stating to the user that it got updated and need to be ran again.

In our case, we're only shipping Linux and Darwin, targeting amd64 platform.

Bucket will have the following structure:

```
mybucket/
  mytool/
	VERSION
	mytool-linux-amd64
	mytool-darwin-amd64
```

## Usage

Updates are easier to deal with when done through a continuous integration platform. We're using CircleCI but the following
excerpt can easily be adapted to whichever solution being used.

### CircleCI

[xgo](https://github.com/karalabe/xgo) is being used to easily cross-compile code.

Adding the following at the end of the build script will push the binaries and its version to S3.

```sh
xgo --targets="linux/amd64,darwin/amd64" -ldflags="-X main.Version=$CIRCLE_BUILD_NUM" .

if [[ "$CIRCLE_BRANCH" = "master" ]]; then
	aws s3 cp mytool-darwin-10.6-amd64 s3://mybucket/mytool/mytool-darwin-amd64 --acl authenticated-read
	aws s3 cp mytool-linux-amd64 s3://mybucket/mytool/mytool-linux-amd64 --acl authenticated-read
	echo -n $CIRCLE_BUILD_NUM > VERSION && aws s3 cp VERSION  s3://mybucket/mytool/VERSION --acl authenticated-read
fi
```

### Example

```go
package main

import (
	"github.com/heetch/s3update"
)

var (
	// This gets set during the compilation. See below.
	Version = ""
)

func main() {
	err := s3update.AutoUpdate(s3update.Updater{
		CurrentVersion: Version,
		S3Bucket:       "mybucket",
		S3Region:       "eu-west-1",
		S3ReleaseKey:   "mytool/mytool-{{OS}}-{{ARCH}}",
		S3VersionKey:   "mytool/VERSION",
	})

  if err != nil {
    // ...
  }

  ...
}
```

Binary must be compiled with a flag specifying the new version:

```sh
go build -ldflags "-X main.Version=111" main.go
```

## Contributions

Any contribution is welcomed!

- Open an issue if you want to discuss bugs/features
- Open Pull Requests if you want to contribute to the code

## Copyright

Copyright © 2016 Heetch

See the [LICENSE](https://github.com/heetch/s3update/blob/master/LICENSE) (MIT) file for more details.



