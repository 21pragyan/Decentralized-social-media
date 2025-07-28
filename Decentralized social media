// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

/**
 * @title Decentralized Social Media Platform
 * @dev A blockchain-based social media platform with posts, following, and tipping functionality
 */
contract Project {
    
    // State variables
    address public owner;
    uint256 public totalPosts;
    uint256 public totalUsers;
    
    // Structs
    struct Post {
        uint256 id;
        address author;
        string content;
        uint256 timestamp;
        uint256 tips;
        bool exists;
    }
    
    struct User {
        address userAddress;
        string username;
        string bio;
        uint256 postCount;
        uint256 followersCount;
        uint256 followingCount;
        bool exists;
    }
    
    // Mappings
    mapping(uint256 => Post) public posts;
    mapping(address => User) public users;
    mapping(address => string) public addressToUsername;
    mapping(string => address) public usernameToAddress;
    mapping(address => mapping(address => bool)) public isFollowing;
    mapping(uint256 => address[]) public postTippers;
    
    // Events
    event UserRegistered(address indexed user, string username);
    event PostCreated(uint256 indexed postId, address indexed author, string content);
    event UserFollowed(address indexed follower, address indexed following);
    event UserUnfollowed(address indexed follower, address indexed unfollowing);
    event PostTipped(uint256 indexed postId, address indexed tipper, uint256 amount);
    
    // Modifiers
    modifier onlyRegistered() {
        require(users[msg.sender].exists, "User not registered");
        _;
    }
    
    modifier validPost(uint256 _postId) {
        require(_postId > 0 && _postId <= totalPosts, "Invalid post ID");
        require(posts[_postId].exists, "Post does not exist");
        _;
    }
    
    constructor() {
        owner = msg.sender;
        totalPosts = 0;
        totalUsers = 0;
    }
    
    /**
     * @dev Core Function 1: Register a new user with username and bio
     * @param _username Unique username for the user
     * @param _bio User's biography/description
     */
    function registerUser(string memory _username, string memory _bio) public {
        require(!users[msg.sender].exists, "User already registered");
        require(bytes(_username).length > 0 && bytes(_username).length <= 32, "Invalid username length");
        require(usernameToAddress[_username] == address(0), "Username already taken");
        
        // Create new user
        users[msg.sender] = User({
            userAddress: msg.sender,
            username: _username,
            bio: _bio,
            postCount: 0,
            followersCount: 0,
            followingCount: 0,
            exists: true
        });
        
        // Update mappings
        addressToUsername[msg.sender] = _username;
        usernameToAddress[_username] = msg.sender;
        totalUsers++;
        
        emit UserRegistered(msg.sender, _username);
    }
    
    /**
     * @dev Core Function 2: Create a new post
     * @param _content The content of the post
     */
    function createPost(string memory _content) public onlyRegistered {
        require(bytes(_content).length > 0 && bytes(_content).length <= 500, "Invalid content length");
        
        totalPosts++;
        
        // Create new post
        posts[totalPosts] = Post({
            id: totalPosts,
            author: msg.sender,
            content: _content,
            timestamp: block.timestamp,
            tips: 0,
            exists: true
        });
        
        // Update user's post count
        users[msg.sender].postCount++;
        
        emit PostCreated(totalPosts, msg.sender, _content);
    }
    
    /**
     * @dev Core Function 3: Follow/Unfollow users and tip posts
     * @param _userToFollow Address of user to follow
     */
    function followUser(address _userToFollow) public onlyRegistered {
        require(_userToFollow != msg.sender, "Cannot follow yourself");
        require(users[_userToFollow].exists, "User to follow does not exist");
        require(!isFollowing[msg.sender][_userToFollow], "Already following this user");
        
        // Update following status
        isFollowing[msg.sender][_userToFollow] = true;
        users[msg.sender].followingCount++;
        users[_userToFollow].followersCount++;
        
        emit UserFollowed(msg.sender, _userToFollow);
    }
    
    /**
     * @dev Unfollow a user
     * @param _userToUnfollow Address of user to unfollow
     */
    function unfollowUser(address _userToUnfollow) public onlyRegistered {
        require(isFollowing[msg.sender][_userToUnfollow], "Not following this user");
        
        // Update following status
        isFollowing[msg.sender][_userToUnfollow] = false;
        users[msg.sender].followingCount--;
        users[_userToUnfollow].followersCount--;
        
        emit UserUnfollowed(msg.sender, _userToUnfollow);
    }
    
    /**
     * @dev Tip a post with Ether
     * @param _postId ID of the post to tip
     */
    function tipPost(uint256 _postId) public payable onlyRegistered validPost(_postId) {
        require(msg.value > 0, "Tip amount must be greater than 0");
        require(posts[_postId].author != msg.sender, "Cannot tip your own post");
        
        // Transfer tip to post author
        address payable author = payable(posts[_postId].author);
        author.transfer(msg.value);
        
        // Update post tips
        posts[_postId].tips += msg.value;
        postTippers[_postId].push(msg.sender);
        
        emit PostTipped(_postId, msg.sender, msg.value);
    }
    
    // View functions
    function getPost(uint256 _postId) public view validPost(_postId) returns (
        uint256 id,
        address author,
        string memory content,
        uint256 timestamp,
        uint256 tips,
        string memory authorUsername
    ) {
        Post memory post = posts[_postId];
        return (
            post.id,
            post.author,
            post.content,
            post.timestamp,
            post.tips,
            addressToUsername[post.author]
        );
    }
    
    function getUser(address _userAddress) public view returns (
        address userAddress,
        string memory username,
        string memory bio,
        uint256 postCount,
        uint256 followersCount,
        uint256 followingCount
    ) {
        require(users[_userAddress].exists, "User does not exist");
        User memory user = users[_userAddress];
        return (
            user.userAddress,
            user.username,
            user.bio,
            user.postCount,
            user.followersCount,
            user.followingCount
        );
    }
    
    function isUserFollowing(address _follower, address _following) public view returns (bool) {
        return isFollowing[_follower][_following];
    }
    
    function getPostTippers(uint256 _postId) public view validPost(_postId) returns (address[] memory) {
        return postTippers[_postId];
    }
    
    function getPlatformStats() public view returns (uint256, uint256) {
        return (totalUsers, totalPosts);
    }
}
