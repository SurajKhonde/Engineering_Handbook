# system Design 

## Functional Requirements:

### design Instgram 

Functional Requirements:

1. Users should be able to create an account and login

2. Users should be able to:
   - Upload posts (image/video + caption)
   - Delete or edit posts

3. Users should be able to:
   - Follow/unfollow other users
   - View profile of users

4. Users should be able to:
   - View a feed of posts from followed users
   - Like and comment on posts

5. System should return:
   - Post details (media, caption, likes, comments)
   - Feed as paginated list of posts
   - Status responses for actions (like, follow, post)

6. Optional features:
   - Notifications
   - Stories / reels

#### Functional Requirment

1. Core Features
2. System Should Allow 
   1. What actions system must support?
   2. User should be able to:
      1. Create account / login
      2. Upload media (image/video)
      3. Delete post
      4. Edit caption (optional)
      5. View user profile
      6. Search users (basic)
3. System Should Return
    1.Think:
    “API kya return karega?”
    Examples:
    - After posting:
    - postId, mediaUrl, timestamp
    - hen fetching feed:
    - List of posts:
       - postId
       - user info
       - media URL
       - likes count
       - comments count
    - After like:
       - success status / updated like count
    - After follow:
       - success status
4. Read vs Write Behavior (still functional level hint)
    Users should be able to:
    - Scroll feed (pagination / infinite scroll)
    - Load posts in chunks

👉 This is functional but hints scaling later  

### design whatsapp 

1. Users should be able to:
   - Register/login using phone number
   - View contacts and user profiles

2. Users should be able to:
   - Send real-time messages (1:1 and group)
   - Share media (images, videos, files)

3. Users should be able to:
   - Create groups
   - Add/remove members
   - Assign admin roles

4. Users should be able to:
   - View chat history with pagination
   - See message status (sent, delivered, read)
   - See online/last seen status

5. System should:
   - Deliver messages in real-time using WebSockets
   - Store messages for future retrieval

6. System should return:
   - Message details (id, content, timestamp, status)
   - Chat history (paginated messages)
   - Status updates (delivery/read)

### design Ecommorce

Functional Requirements:

1. Users should be able to:
   - Signup/login using email or phone
   - Manage profile, addresses, and payment methods

2. Buyers should be able to:
   - Browse/search products
   - View product details (images, videos, price, description)
   - Add/remove items from cart and wishlist
   - Place orders and make payments
   - View order history and track order status

3. Sellers should be able to:
   - Add/update/delete products
   - Manage inventory

4. Users should be able to:
   - Read and write reviews and ratings

5. System should:
   - Show product availability
   - Fetch personalized data (cart, wishlist)
   - Update inventory after order

6. System should return:
   - Product list (paginated)
   - Cart details and totals
   - Order details (id, status, payment info)
   - Success/failure responses for actions

### design Music App

  Functional Requirements:

1. Users should be able to:
   - Search songs, albums, and artists
   - Play, pause, and skip songs

2. Users should be able to:
   - Create and manage playlists
   - Like and save songs

3. Users should be able to:
   - Browse albums, artists, and genres
   - Discover trending and recommended content

4. Users should be able to:
   - Signup/login and manage subscription

5. System should:
   - Stream audio efficiently with buffering
   - Adjust quality based on network

6. System should return:
   - Audio stream and metadata for playback
   - Search results (paginated)
   - Playlist and library data

7. Optional:
   - Offline download
   - Lyrics, podcasts, sharing
