### Queries

1. create a follow request

   ```sql
    insert into follow_request (follower_id, content_creator_id, request_status)
    select <current_user_id> , <content_creator_id>,'pending'
    from generate_series(1,1) -- this is a dummy table
    where exists (select user_id from user_profile where id = <content_creator_id> and visibility = 'private')
   ```

2. upgrade a follow request into follower
   ```sql
       BEGIN
           insert into follow(follower_id, content_creator_id) values (<current_id>, <content_creator_id>)
           update follow_request set request_status = 'accepted' where follower_id = <current_user_id> and content_creator_id = <content_creator_id>
       COMMIT
   ```
3. create an asset - a normal insert query
4. create a post | create an asset

   ```sql
       -- input is a list of assets, user_id, description, tag
       insert into asset (asset_type, owner_id, uri) values (<asset_type>, <user_id>, <uri>)
       -- application logic will store the asset_ids here for insert
       -- transaction is not required as image uploads are invoved and an asset could may be skipped or not created
       -- in such scenarios , application logic should handle prompting the user to proceed or retry.

        BEGIN
            insert into post values (owner_id, description, like_count) values (<user_id>, <description>, 0)
            -- for each asset
            insert into post_asset (post_id, asset_id, seq) values (<post_id>, <asset_id>, <array_index>)
       COMMIT

       BEGIN
       -- if tag does not exist
            insert into tag (description) values (<tag>)
            insert into post_tag (post_id, tag_id) values (post_id, tag_id)
       COMMIT
   ```

5. create a user story flow is similar. It has equivalent flow.
6. reordering image sequence in the post is a messy operation as the sequence of a many rows would need to be updated. It is the same as array vs linkedlist
   problem and can be handled similarly. This is an infrequent operation so it may be overkill to do it the linked list
7. get users by handle or name - will use full text search here
   ```sql
       select user_id,
           ts_rank(to_tsvector (handle || ' ' || first_name || ' ' || last_name), websearch_to_tsquery(<search_text>)) match_score
       from user_profile
       where to_tsvector (handle || ' ' || first_name || ' ' || last_name) @@websearch_to_tsquery(<search_text>)
       order by match_score
   ```
8. get posts by hash tag
   ```sql
       select pt.post_id from post_tag pt join tag t on pt.tag_id = t.tag_id
           where t.description = <hash_text>
   ```
9. like a post
   ```sql
       insert into user_post_like (user_id, post_id) values(<current_user_id>, <post_id>);
   ```
10. create a hashtag
    ```sql
        insert into tag(description) values(<description>)
    ```
11. create a comment
    ```sql
        insert into comment values(content, owner_id, parent_id, post_id) values (<content>, <user_id>, <parent_id>, <post_id>);
    ```
12. get all content creators that the user follows

    ```sql
        select u.user_id from
            user_profile u join follower f on u.user_id = f.follower_id
        where u.user_id = <current_user_id>
    ```

13. get all assets of the post. These are ordered by asset squence.

    ```sql
        select
            p.post_id,
            jsonb_agg(jsonb_build_object('assetId', a.asset_id, 'assetType', a.assetType, 'uri', a.uri, 'sequence': pa.sequence), ORDER BY pa.sequence) asset_arr
            from post p join post_asset pa on p.post_id = pa.post_id
                join asset a on pa.asset_id = a.asset_id
            where p.post_id = <post_id>
            group_by p.post_id
    ```

14. get all tags of a post

    ```sql
        select p.post_id
        from post p join post_tag pt on p.post_id = pt.post_id
        join tag t on pt.tag_id = t.tag_id
        where p.post_id = <post_id>
    ```

15. get all comments of a post

    ```sql
        select content, comment_id, owner_id, parent_id
        from comment
        where post_id = <post_id>
    ```

16. get all posts that the user has liked
    ```sql
        select * from user_post_like
            where user_id  = <user_id>
    ```
17. get

    - posts of all the firends of the user
    - sorted by descending time.
    - include the assets.
    - first 10 comments,
    - hashtags.
    - and wether the post has been liked by current user

    ```sql
        with cte_creators_of_user as (
            select u.user_id from
            user_profile u join follower f on u.user_id = f.follower_id
        where u.user_id = <current_user_id>)

        with cte_posts_user_content_creators as (
            select p.post_id, p.description, p.created_at
                from post p join user_profile u on p.owner_id = u.user_id
                where <user_id> in (content_creators_of_user)
                order by p.created_at desc
        )

        with cte_user_post_creator_tag as (
            select p.post_id, p.description, p.created_at 
                jsonb_aggr(jsonb_build_object('tag_id', pt.tag_id, 'description', pt.description)) as tags
            from cte_posts_user_content_creators p join post_tag pt on p.post_id = pt.post_id,
            group by p.post_id
        )

        -- todo update this query to get the first 5 comments 3 comments only.
        with cte_user_post_creator_tag_comment as (
            select p.post_id, p.description, p.tags, p.created_at, 
                jsonb_aggr(jsonb_build_object('comment_id', c.comment_id, 'content': c.content, parent_id: c.parent_id, )) as posts
            from cte_user_post_creator_tag p join comment c on p.post_id = c.post_id
            group by p.post_id
        )

        with cte_user_creator_posts_tag_comment_assets as (
            select
            p.post_id,
            p.description, 
            p.comments, 
            p.tags,
            MIN(p.created_at) created_at -- using min as not aggregating by timestamp so using an aggregation fn
            jsonb_agg(jsonb_build_object('assetId', a.asset_id, 'assetType', a.assetType, 'uri', a.uri, 'sequence': pa.sequence), ORDER BY pa.sequence) asset_arr
            from post cte_posts_user_content_creators p join post_asset pa on p.post_id = pa.post_id
                join asset a on pa.asset_id = a.asset_id
            where p.post_id = <post_id>
            group_by p.post_id
        )

        -- this will be used with limit and offset
    ```
