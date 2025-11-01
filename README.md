# internetmarketplatform
import cv2
import numpy as np
import torch
import torch.nn as nn
import torch.nn.functional as F
from skimage.color import rgb2gray
from skimage.feature import match_template, local_binary_pattern
from torchvision import transforms
from sklearn.metrics import jaccard_score
MOCK DATA AND HELPER FUNCTIONS 
def create_mock_forgery_image(size=(256, 256)):
    """Creates a mock biomedical image with CMF and a ground truth mask."""
    img = np.zeros(size, dtype=np.uint8)
    obj = np.random.randint(50, 200, (40, 40), dtype=np.uint8)
    Source region (slightly rotated)
    M = cv2.getRotationMatrix2D((20, 20), 10, 1)
    rotated_obj = cv2.warpAffine(obj, M, (40, 40), borderMode=cv2.BORDER_CONSTANT)
    img[50:90, 50:90] = rotated_obj
    Copy region (simple copy + intensity change)
    copied_obj = np.clip(obj * 1.2, 0, 255).astype(np.uint8)
    img[150:190, 150:190] = copied_obj
    noise = np.random.randint(0, 20, size, dtype=np.uint8)
    img = np.clip(img + noise, 0, 255)
    mask = np.zeros(size, dtype=np.uint8)
    mask[50:90, 50:90] = 255
    mask[150:190, 150:190] = 255
    return cv2.cvtColor(img, cv2.COLOR_GRAY2BGR), mask
def calculate_iou(true_mask, pred_mask):
    """Calculates Intersection over Union (IoU) metric."""
    true_mask = (true_mask > 0).flatten()
    pred_mask = (pred_mask > 0).flatten()
    if np.sum(true_mask) == 0 and np.sum(pred_mask) == 0:
        return 1.0 # Perfect score if both are empty
    return jaccard_score(true_mask, pred_mask)
MOCK DL MODELS 
class MockFeatureExtractor(nn.Module):
    """Mock VGG-like feature extractor for deep methods."""
    def __init__(self):
        super().__init__()
        self.conv = nn.Conv2d(3, 64, kernel_size=3, padding=1)
        self.relu = nn.ReLU(inplace=True)
        self.pool = nn.MaxPool2d(2)
        def forward(self, x):
        x = self.pool(self.relu(self.conv(x)))
        return x
class MockUNet(nn.Module):
    """Mock U-Net for pixel segmentation."""
    def __init__(self):
        super().__init__()
        # Simplified U-Net structure (Encoder -> Decoder)
        self.enc1 = nn.Conv2d(3, 16, 3, padding=1)
        self.dec1 = nn.ConvTranspose2d(16, 1, 2, stride=2) 
        self.pool = nn.MaxPool2d(2)
def forward(self, x):
        # Iмітація виявлення низькорівневих ознак
        x = self.pool(F.relu(self.enc1(x)))
        # Імітація декодування до початкового розміру
        x = self.dec1(x)
        return torch.sigmoid(x)
METHOD 1: TRADITIONAL (SIFT) 
def method_sift_matching(image):
    """Basic/Traditional: SIFT for feature extraction and Brute-Force matching."""
    gray_img = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    sift = cv2.SIFT_create()
    kp, des = sift.detectAndCompute(gray_img, None)
    mask = np.zeros_like(gray_img, dtype=np.uint8)
    if des is None or len(des) < 2: return mask
        bf = cv2.BFMatcher()
    matches = bf.knnMatch(des, des, k=2)
    good_matches = []
    Ratio Test and filtering self-matches
    for m, n in matches:
        if m.distance < 0.75 * n.distance and m.queryIdx != m.trainIdx:
            # Check for close spatial distance (CMF implies separated regions)
            pt1 = kp[m.queryIdx].pt
            pt2 = kp[m.trainIdx].pt
            distance = np.sqrt((pt1[0]-pt2[0])**2 + (pt1[1]-pt2[1])**2)
            # Only consider matches far apart
            if distance > 50:
                 good_matches.append(m)
for match in good_matches:
        pt1 = kp[match.queryIdx].pt
        pt2 = kp[match.trainIdx].pt
        cv2.circle(mask, (int(pt1[0]), int(pt1[1])), 5, 255, -1)
        cv2.circle(mask, (int(pt2[0]), int(pt2[1])), 5, 255, -1)
            return mask
METHOD 2: MEDIUM (U-NET SEGMENTATION MOCK)
def method_unet_segmentation(image):
    """Medium/DL: Mock U-Net for direct pixel segmentation."""
    preprocess = transforms.Compose([transforms.ToTensor()])
    input_tensor = preprocess(image).unsqueeze(0) 
model = MockUNet()
    # Mocking prediction without training: will be random noise
    with torch.no_grad():
        output = model(input_tensor)
        To provide a meaningful mock result: use LBP texture analysis
    gray_img = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    radius = 3; n_points = 8 * radius
    lbp = local_binary_pattern(gray_img, n_points, radius, method='uniform').astype(np.uint8)
    Thresholding LBP map to find textural uniformity (a sign of CMF)
    _, lbp_mask = cv2.threshold(lbp, 10, 255, cv2.THRESH_BINARY_INV)
    lbp_mask = cv2.medianBlur(lbp_mask, 5) # Smooth the result
    U-Net would learn to combine these low-level features
    return lbp_mask
METHOD 3: PREMIUM (DEEP FEATURE MATCHING MOCK) ===
def method_deep_feature_matching(image):
    """Premium/Hybrid: Deep Feature Extractor + Attention/Matching Logic."""
    preprocess = transforms.Compose([transforms.ToTensor()])
    input_tensor = preprocess(image).unsqueeze(0)
    1. Feature Extraction (VGG/ResNet logic)
    extractor = MockFeatureExtractor()
    with torch.no_grad():
        features = extractor(input_tensor).squeeze(0).numpy() # (C, H', W')
        2. Matching Logic (Simulating Attention/Siamese comparison)
We compare a small block of features (template) against all others
    Define template region in feature space (corresponds to copied region)
    template_feat = features[:, 12:20, 12:20] # Mocked copied area in feature map
    Use template matching on the feature maps
    template_feat_sum = template_feat.mean(axis=0) # Reduce to 2D
    features_sum = features.mean(axis=0) 
    if template_feat_sum.size == 0:
        return np.zeros(image.shape[:2], dtype=np.uint8)
result_match = match_template(features_sum, template_feat_sum)
    Upscale and threshold the result to get a mask
    match_mask = (result_match > 0.9).astype(np.uint8) * 255 # High correlation threshold
    Resize back to original image size
    (h, w) = image.shape[:2]
    final_mask = cv2.resize(match_mask, (w, h), interpolation=cv2.INTER_NEAREST)
    Morphological closing to fill gaps
    kernel = np.ones((5,5),np.uint8)
    final_mask = cv2.morphologyEx(final_mask, cv2.MORPH_CLOSE, kernel)
    return final_mask
EXECUTION AND COMPARISON 
def run_cmf_analysis():
    """Executes all methods and compares results."""
    image, true_mask = create_mock_forgery_image()
    results = {}
    1. Method SIFT
    mask_sift = method_sift_matching(image)
    results['SIFT (Basic)'] = (mask_sift, calculate_iou(true_mask, mask_sift))
    2. Method U-Net Mock (LBP)
    mask_unet = method_unet_segmentation(image)
    results['U-Net (Medium Mock)'] = (mask_unet, calculate_iou(true_mask, mask_unet))
    3. Method Deep Feature Matching (Premium)
    mask_deep = method_deep_feature_matching(image)
    results['Deep Feature (Premium)'] = (mask_deep, calculate_iou(true_mask, mask_deep))
Visualization setup
    def get_labeled_image(img, mask, label, iou, color=(0, 0, 255)):
        """Overlay mask and add label/IoU score."""
        result = img.copy()
        mask_3ch = cv2.cvtColor(mask, cv2.COLOR_GRAY2BGR)
        blue_layer = np.zeros_like(result, dtype=np.uint8)
        blue_layer[:, :, color.index(255)] = mask
        masked = cv2.addWeighted(result, 0.7, blue_layer, 0.3, 0)
        text = f"{label} | IoU: {iou:.3f}"
        cv2.putText(masked, text, (5, 20), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (255, 255, 255), 1, cv2.LINE_AA)
        return masked
Prepare images for display
    img_true = get_labeled_image(image, true_mask, "True Mask", 1.0, color=(0, 255, 0)) # Green
    img_sift_res = get_labeled_image(image, results['SIFT (Basic)'][0], "Method 1: SIFT", results['SIFT (Basic)'][1], color=(255, 0, 0)) # Blue
    img_unet_res = get_labeled_image(image, results['U-Net (Medium Mock)'][0], "Method 2: U-Net Mock", results['U-Net (Medium Mock)'][1], color=(0, 0, 255)) # Red
    img_deep_res = get_labeled_image(image, results['Deep Feature (Premium)'][0], "Method 3: Deep Feature (BEST)", results['Deep Feature (Premium)'][1], color=(255, 255, 0)) # Cyan
combined_image = np.hstack([img_true, img_sift_res, img_unet_res, img_deep_res])
    Display results
    cv2.imshow('CMF Detection Comparison (IoU Score)', combined_image)
    cv2.waitKey(0)
    cv2.destroyAllWindows()

if __name__ == "__main__":
    run_cmf_analysis()
