from flask import Flask, request, jsonify
import joblib
import re
import tldextract

# Load trained model
model = joblib.load("zyqox_model.pkl")

app = Flask(__name__)

# Example feature extractor (adjust based on how you trained your model)
def extract_features(url):
    features = []
    features.append(1 if re.search(r'\d{1,3}(\.\d{1,3}){3}', url) else 0)  # IP in URL
    features.append(1 if "@" in url else 0)  # @ symbol
    features.append(len(url))  # Length of URL
    features.append(url.count('-'))  # Hyphen count
    features.append(url.count('.'))  # Dot count
    ext = tldextract.extract(url)
    features.append(len(ext.subdomain))  # Subdomain length
    return [features]

@app.route("/predict", methods=["POST"])
def predict():
    data = request.json
    url = data.get("url", "")

    if not url:
        return jsonify({"error": "No URL provided"}), 400

    # Extract features
    features = extract_features(url)

    # Predict with model
    prediction = model.predict(features)[0]

    return jsonify({
        "url": url,
        "prediction": prediction
    })

@app.route("/", methods=["GET"])
def home():
    return "🛡️ Zyqox API is running! Use /predict to test URLs."

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
