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
4. create a post

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
9. 
