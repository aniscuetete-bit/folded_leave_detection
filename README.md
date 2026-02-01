% MATLAB code for image preprocessing:
image = imread('C:\Users\anisc\OneDrive\Desktop\DSC_6847.JPG');
figure, imshow(image); title('Original Image');
hsvImage = rgb2hsv(image);
figure, imshow(hsvImage); title('HSV Image');
H = hsvImage(:,:,1);
S = hsvImage(:,:,2);
V = hsvImage(:,:,3);
figure, imshow(H); title('Hue Channel');
figure, imshow(S); title('Saturation Channel');
figure, imshow(V); title('Value Channel');
leafMask = (H > 0.2 & H < 0.4) & (S > 0.1) & (V > 0.3);
figure, imshow(leafMask); title('Initial Leaf Mask');
leafMask = bwareaopen(leafMask, 500);  
leafMask = imfill(leafMask, 'holes');  
figure, imshow(leafMask); title('Cleaned Leaf Mask');
blackBackground = zeros(size(image), 'uint8');
outputImage = blackBackground;

for c = 1:3  % Apply mask to each color channel
    channel = image(:,:,c);
    channel(~leafMask) = 0;
    outputImage(:,:,c) = channel;
end
figure, imshow(outputImage); title('Pre-Processed Image (Leaf Regions Only)');
outputPath = 'C:\Users\anisc\OneDrive\Desktop\Savedpicture\Processed_P1R1.jpg';
imwrite(outputImage, outputPath);
disp(['Processed image saved to: ', outputPath]);

% MATLAB code for fipped leaf detectio
img = imread('C:\Users\anisc\OneDrive\Desktop\Savedpicture\Processed_P1R1.jpg'); 
figure, imshow(img), title('Original Image');
grayImg = rgb2gray(img);
figure, imshow(grayImg), title('Grayscale Image');
grayImg = imadjust(grayImg);
figure, imshow(grayImg), title('Contrast-Enhanced Grayscale Image');
numRows = 40; 
numCols = 40; 
blockHeight = floor(size(grayImg, 1) / numRows);
blockWidth = floor(size(grayImg, 2) / numCols);
intensityThreshold = 235; 
sizeThreshold = 100; 
disp(['Block size: ', num2str(blockHeight), 'x', num2str(blockWidth)]);
boundingBoxes = [];
blockCounter = 0;
for i = 1:numRows
    for j = 1:numCols
        rowStart = (i - 1) * blockHeight + 1;
        rowEnd = min(i * blockHeight, size(grayImg, 1));
        colStart = (j - 1) * blockWidth + 1;
        colEnd = min(j * blockWidth, size(grayImg, 2));

        block = grayImg(rowStart:rowEnd, colStart:colEnd);
        meanIntensity = mean(block(:));

        if meanIntensity > intensityThreshold
            blockArea = (rowEnd - rowStart + 1) * (colEnd - colStart + 1);
            if blockArea > sizeThreshold
                blockCounter = blockCounter + 1;
                boundingBoxes = [boundingBoxes; colStart, rowStart, colEnd - colStart, rowEnd - rowStart];
                disp(['Block ', num2str(blockCounter), ': Mean Intensity = ', num2str(meanIntensity), ...
                      ', Area = ', num2str(blockArea)]);
            end
        end
    end
end

disp(['Total High-Intensity Blocks: ', num2str(blockCounter)]);
tempImage = insertShape(img, 'Rectangle', boundingBoxes, 'Color', 'yellow', 'LineWidth', 1);
figure, imshow(tempImage), title('Detected High-Intensity Blocks');
finalBoundingBoxes = mergeClosestBoundingBoxes(boundingBoxes, 100);
disp(['Merged bounding boxes: ', num2str(size(finalBoundingBoxes, 1))]);
blockSize = 30;
for i = 1:size(finalBoundingBoxes, 1)
    bbox = finalBoundingBoxes(i, :);
    centerX = bbox(1) + bbox(3) / 2;
    centerY = bbox(2) + bbox(4) / 2;
    xStart = round(centerX - blockSize / 2);
    yStart = round(centerY - blockSize / 2);
    xStart = max(1, min(xStart, size(img, 2) - blockSize));
    yStart = max(1, min(yStart, size(img, 1) - blockSize));
    finalBoundingBoxes(i, :) = [xStart, yStart, blockSize, blockSize];
end

disp('Standardized final bounding boxes created.');
outputImage = insertShape(img, 'Rectangle', finalBoundingBoxes, 'Color', 'red', 'LineWidth', 2);
numFoldedLeaves = size(finalBoundingBoxes, 1);
disp(['Final number of folded leaves detected: ', num2str(numFoldedLeaves)]);
figure, imshow(outputImage), title(['Detected Folded Leaves: ', num2str(numFoldedLeaves)]);
outputFolder = 'C:\Users\anisc\OneDrive\Desktop\Adnan remaining';
outputFilename = 'P130R2_F.jpg';
imwrite(outputImage, fullfile(outputFolder, outputFilename));
disp('Output image saved.');


