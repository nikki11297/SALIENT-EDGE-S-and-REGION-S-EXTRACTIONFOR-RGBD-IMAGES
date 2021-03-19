clear all;
close all;
clc;
img=imread('D:\Documents\Final Year Project\RGBD\result1\s146.jpg'); %reading RGB image
%depth=im.smoothedDepth;
depth=imread('D:\Documents\Final Year Project\RGBD\result1\s146.png'); %reading depth image which is in png format
newim= uint8(255*mat2gray(depth));
figure;imshow(newim);
figure;imshow(img);
[C,D] = superpixels(depth,18500);
finalimage = zeros(size(depth),'like',depth);
finalimage1=zeros(size(depth),'like',depth);
idnx = label2idx(C);
BAW = boundarymask(C);
imshow(imoverlay(newim,BAW,'cyan'),[]);
nRows = size(depth,1);
nCols = size(depth,2);
for labelValue = 1:D
   memberPixelIdx1 = idnx{labelValue};
   finalimage(memberPixelIdx1) = ((newim(memberPixelIdx1)) - min(newim(memberPixelIdx1))).^2;
   finalimage1(memberPixelIdx1) = ((newim(memberPixelIdx1)) - mean(newim(memberPixelIdx1))).^2;
  end  
figure
imshow(newim(memberPixelIdx1==1))  
imshow(finalimage,[])
figure
imshow(finalimage1,[]);
out=finalimage1+finalimage; % combing min and mean of the images
figure;imshow(out,[]);title('combing min and mean of the images');
dilatedImage = imdilate(out,strel('disk',2));
figure;imshow(dilatedImage,[]);title('dilatedimage of min and mean');
can=edge(depth,'canny');% applying canny to the depth image
figure;imshow(can,[]);title('canny');
out1=can.*double(dilatedImage); %combining canny and out
out1=uint8(255*mat2gray(out1));
figure;imshow(out1);title('combining canny and out');
CC=bwconncomp(out1); %finding the largest connected component
 numOfPixels = cellfun(@numel,CC.PixelIdxList);
 [unused,indexOfMax] = max(numOfPixels);
 biggest = zeros(size(out1));
 biggest(CC.PixelIdxList{indexOfMax}) = 1;
figure;imshow(biggest);title('final salient edge');
dilatedImage = imdilate(biggest,strel('disk',2));
 figure;imshow(dilatedImage,[]);title('dilated final salient edge');

A=GetMC(img); % getting mc of the RGB image
 A1 = imresize(A,size(depth));
 figure;imshow(A1);
 Y=(max(A1(:))); %getting max of the mc image

hliml = zeros(1,size(biggest,1)); %left pointer
hlimr = zeros(1,size(biggest,1)); %right pointer

for i = 1:size(biggest,1)
    if ~isempty(find([diff(biggest(i,:)),0]==1,1,'first'))  %traversing from left
hliml(i) = find([diff(biggest(i,:)),0]==1,1,'first');
    end

    if ~isempty(find([diff(biggest(i,:)),0]==-1,1,'last')) %traversing from right
hlimr(i) = find([diff(biggest(i,:)),0]==-1,1,'last');
    end
end


for i = 1:size(biggest,1)
    if hliml(i)~=0
        biggest(i,hliml(i):hlimr(i)) = Y; %fiing the edge map with maximum of salient value of MC image
    end
end

figure;imshow(biggest);title('final saliency map');
