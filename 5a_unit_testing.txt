import unittest
import requests
import json

# Mock requests.post to simulate Watson NLP response when offline or on a local machine
original_post = requests.post
def mock_post(url, *args, **kwargs):
    if 'sn-watson-emotion.labs.skills.network' in url:
        class MockResponse:
            def __init__(self, text):
                self.text = text
                self.status_code = 200
        
        payload = kwargs.get('json', {})
        text = payload.get('raw_document', {}).get('text', '')
        
        if "glad" in text:
            emotions = {"anger": 0.01, "disgust": 0.01, "fear": 0.01, "joy": 0.96, "sadness": 0.01}
        elif "mad" in text:
            emotions = {"anger": 0.96, "disgust": 0.01, "fear": 0.01, "joy": 0.01, "sadness": 0.01}
        elif "disgusted" in text:
            emotions = {"anger": 0.01, "disgust": 0.96, "fear": 0.01, "joy": 0.01, "sadness": 0.01}
        elif "sad" in text:
            emotions = {"anger": 0.01, "disgust": 0.01, "fear": 0.01, "joy": 0.01, "sadness": 0.96}
        elif "afraid" in text:
            emotions = {"anger": 0.01, "disgust": 0.01, "fear": 0.96, "joy": 0.01, "sadness": 0.01}
        else:
            emotions = {"anger": 0.0, "disgust": 0.0, "fear": 0.0, "joy": 0.0, "sadness": 0.0}
            
        return MockResponse(json.dumps({"emotionPredictions": [{"emotion": emotions}]}))
    return original_post(url, *args, **kwargs)

requests.post = mock_post

from EmotionDetection import emotion_detector

class TestEmotionDetector(unittest.TestCase):
    def test_joy(self):
        result = emotion_detector("I am glad this happened")
        self.assertEqual(result['dominant_emotion'], 'joy')

    def test_anger(self):
        result = emotion_detector("I am really mad about this")
        self.assertEqual(result['dominant_emotion'], 'anger')

    def test_disgust(self):
        result = emotion_detector("I feel disgusted just hearing about this")
        self.assertEqual(result['dominant_emotion'], 'disgust')

    def test_sadness(self):
        result = emotion_detector("I am so sad about this")
        self.assertEqual(result['dominant_emotion'], 'sadness')

    def test_fear(self):
        result = emotion_detector("I am really afraid that this will happen")
        self.assertEqual(result['dominant_emotion'], 'fear')

if __name__ == '__main__':
    unittest.main()
