# -Using-IBM-Watson-NLU-and-TwitterKit-to-create-Honey-s-Social-Media-Analyzer-hSOMA
integrating both the Natural Language Understanding (NLU) service from IBM Watson and the Twitter API to analyze and extract sentiment, emotions, keywords, and other important information from tweets.
Prerequisites:

    IBM Watson NLU service set up and API credentials.
    Twitter Developer Account and credentials to access the Twitter API.
    Xcode and Swift for iOS development.
    TwitterKit for integrating Twitter functionality in the app.

Steps Overview:

    Authenticate with Twitter using TwitterKit.
    Fetch Tweets using Twitter API.
    Send Tweets to IBM Watson NLU for analysis.
    Display analysis results in the app.

Step-by-Step Code Implementation:
Step 1: Set up IBM Watson NLU

    Go to the IBM Cloud console.

    Create a new Natural Language Understanding (NLU) service instance.

    Get the API Key and URL from the service instance.

    Add the IBM Watson SDK to your iOS project using CocoaPods:

    pod 'IBMCloudKit', '~> 1.0'
    pod 'WatsonDeveloperCloud', '~> 1.2'

Step 2: Set up Twitter API

    Go to Twitter Developer and create an app.

    Get the API Key, API Secret Key, Access Token, and Access Token Secret.

    Add the TwitterKit SDK to your project.

    Install TwitterKit using CocoaPods:

    pod 'TwitterKit'

Step 3: Implementing the App in Swift

Here’s the Swift code for creating Tanmay's Social Media Analyzer (tSOMA) using IBM Watson NLU and TwitterKit.

import UIKit
import TwitterKit
import WatsonDeveloperCloud
import Alamofire

class ViewController: UIViewController {
    
    @IBOutlet weak var tweetTextView: UITextView!
    @IBOutlet weak var sentimentLabel: UILabel!
    @IBOutlet weak var keywordsLabel: UILabel!
    
    // IBM Watson NLU instance
    let nlu = NaturalLanguageUnderstanding(version: "2021-03-25", apiKey: "YOUR_WATSON_NLU_API_KEY")
    
    // Twitter API keys
    let consumerKey = "YOUR_TWITTER_CONSUMER_KEY"
    let consumerSecret = "YOUR_TWITTER_CONSUMER_SECRET"
    let accessToken = "YOUR_TWITTER_ACCESS_TOKEN"
    let accessTokenSecret = "YOUR_TWITTER_ACCESS_TOKEN_SECRET"
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Set up Twitter authentication
        TWTRTwitter.sharedInstance().start(withConsumerKey: consumerKey, consumerSecret: consumerSecret)
        
        // Example: Fetch the latest tweet from a user's timeline
        fetchTweets(for: "elonmusk")
    }
    
    // Fetch tweets using TwitterKit
    func fetchTweets(for username: String) {
        let client = TWTRAPIClient.withCurrentUser()
        let request = client.urlRequest(withMethod: "GET", urlString: "https://api.twitter.com/1.1/statuses/user_timeline.json?screen_name=\(username)&count=5", parameters: nil, error: nil)
        
        client.sendTwitterRequest(request) { response, data, error in
            if let error = error {
                print("Error fetching tweets: \(error.localizedDescription)")
                return
            }
            
            // Parse the fetched tweets
            if let tweets = try? JSONDecoder().decode([Tweet].self, from: data!) {
                for tweet in tweets {
                    print("Tweet: \(tweet.text)")
                    
                    // Analyze the tweet with Watson NLU
                    self.analyzeTweet(tweet.text)
                }
            }
        }
    }
    
    // Analyze tweet using Watson NLU
    func analyzeTweet(_ tweetText: String) {
        let parameters = ["text": tweetText, "features": ["sentiment": [], "keywords": []]] as [String : Any]
        
        nlu.analyze(parameters: parameters) { response, error in
            if let error = error {
                print("Error analyzing tweet: \(error.localizedDescription)")
                return
            }
            
            // Handle the analysis response
            if let sentiment = response?.sentiment?.document?.label {
                DispatchQueue.main.async {
                    self.sentimentLabel.text = "Sentiment: \(sentiment)"
                }
            }
            
            if let keywords = response?.keywords {
                DispatchQueue.main.async {
                    self.keywordsLabel.text = "Keywords: \(keywords.map { $0.text }.joined(separator: ", "))"
                }
            }
        }
    }
}

// Define a struct to decode the fetched tweets
struct Tweet: Codable {
    var text: String
}

Key Components:

    TwitterKit Setup:
        Using the TwitterKit API, we authenticate and fetch recent tweets from a specified Twitter user's timeline.
        The TWTRAPIClient is used to make the API call to get the user's tweets.

    IBM Watson NLU Setup:
        The Natural Language Understanding API is used to analyze the text of the tweet.
        We request both sentiment analysis and keywords extraction from Watson's NLU API.
        The response contains sentiment scores (positive, neutral, or negative) and keywords found in the tweet.

    Displaying Results:
        Once the sentiment and keywords are extracted, they are displayed on the user interface.

Notes:

    Authentication: Twitter authentication uses OAuth, and you'll need to set up Twitter Developer credentials to interact with Twitter's API. For IBM Watson, use your API key to authenticate requests.

    Data Analysis: The sentiment and keywords are displayed in the app’s interface after analyzing the tweets. You can extend this to add more analysis features like emotions, categories, etc.

    Bluetooth or IoT Integration: While this example doesn't include direct IoT functionality, you can extend the app to trigger IoT actions (e.g., controlling smart devices based on sentiment) using Bluetooth or Wi-Fi protocols.

Next Steps:

    Deploy the app: Once you have the basic functionality working, you can expand the app to include more features, such as analyzing multiple tweets, displaying more detailed sentiment analysis, or integrating with other social media platforms.
    Customize UI: Improve the UI/UX of the app to make it more interactive and user-friendly.

This code demonstrates how to use IBM Watson NLU to analyze tweets fetched via TwitterKit and display useful insights such as sentiment and keywords. You can further enhance the app with more advanced features as needed.
