---
title: "SQLBoilerでJoinする方法"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["golang", "orm", "sqlboiler"]
published: true
---

### 背景

GoのORMであるSQLBoilerで、Joinするのが少し面倒だったので、書きました

### 方法

Joinして取得するデータの構造体を生成してから、Queryを実行する必要があります

ex) `TweetObject`テーブルと`MediaObject`テーブルを`tweetId`フィールドを用いてJoin

```go
type TweetAndMedia struct {
	TweetObject models.TweetObject `boil:"tweet_objects, bind"`
	MediaObject models.MediaObject `boil:"media_objects, bind"`
}
```

```go
func (tr *tweetRepository) test(ctx context.Context, parameter dto.FindTweetParameter) {
	tweetAndMediaList := make([]TweetAndMedia, 0)
	err := models.NewQuery(
		qm.Select("*"),
		qm.From(models.TableNames.TweetObjects),
		qm.LeftOuterJoin("media_objects m on m.tweetId = tweet_objects.tweetId"),
		models.TweetObjectWhere.TweetId.EQ(parameter.Id),
	).Bind(ctx, tr.DB, &tweetAndMediaList)

	if err != nil {
		fmt.Println(err)
	}

	fmt.Println(tweetAndMediaList)
}
```

### 参考

https://thedevelopercafe.com/articles/sql-in-go-with-sqlboiler-ac8efc4c5cb8

https://ken-aio.github.io/post/2019/04/01/golang-sqlboiler-select/#join%E3%81%97%E3%81%9F%E3%83%87%E3%83%BC%E3%82%BF%E3%82%92%E5%8F%96%E5%BE%97
