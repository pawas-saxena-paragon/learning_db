### Entities

1. user_profile
2. follow
3. post
4. comment
5. user_feed
6. asset
7. story

### Excluded

1. messaging
2. not writing CRUD for every thing . Just writing the main queries on each table
3. permissions on post. All users can view the post if they are friends
4. blocking a user. Which is a kind of permission operation
5. recommendation in feed. Feed here is just activites of friends
6. friend recommendations.
7. reels
8. muting updates from a user . part of permissions
9. Analytics for content creators like how the posts are performing
10. moderation and reporting on posts
11. share a post. creating a temporary shareable url
12. deactivate a user
13. liking a comment

### operations

1. create a user
2. ✅ follow a user
3. ✅ accept or deny friend request
4. ✅ create a post
5. create user feed
6. ✅ create an asset
7. ✅create a comment
8. ✅create a reply on comment
9. ✅ create a user story
10. find all stories of the given user
11. ✅ allow users to add hashtags to posts
12. ✅ search posts by hashtags
13. ✅ like a post
14. delete a user along with posts
15. delete a post along with likes and comments and assets
16. ✅ find content_creators by handle or name
17. ✅ create a hash tag
18. get posts of all the firends of the user sorted by descending time. This should also include the assets
19. ✅get all content_creators who the user follows.
20. ✅get all assets of the post order by the asset order
21. ✅get all tags for a post
22. ✅get all the posts that the user has liked
