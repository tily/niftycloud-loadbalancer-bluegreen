# NIFTYCloud LoadBalancer BlueeGreen

ニフティクラウドの LB で blue-green デプロイメントを行うためのスクリプト。

## Usage

	## 旧環境アカウントの API キー
	export DETACH_ACCESS_KEY_ID=XXXXXXXXXXXXXXXXXXXX
	export DETACH_SECRET_ACCESS_KEY=YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY
	## 新環境アカウントの API キー
	export ATTACH_ACCESS_KEY_ID=AAAAAAAAAAAAAAAAAAAA
	export ATTACH_SECRET_ACCESS_KEY=BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
	bundle exec thor bluegreen \
	    --attach-lb-owner abc12345 \
	    --lb-name lb001 \
	    --lb-port 443 \
	    --instance-port 443 \
	    --detach old001 old002 \
	    --attach new001 new002 \
	    --region east-1

## TODO

* 使い方をもっと詳しくドキュメントに書く
