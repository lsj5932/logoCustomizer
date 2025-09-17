// Figma Plugin: Image Size Converter
// 메인 플러그인 코드

// UI 표시
figma.showUI(__html__, { width: 300, height: 500 });

// UI에서 오는 메시지 처리
figma.ui.onmessage = async (msg) => {
    console.log("📨 메시지 수신:", msg.type);
    
    switch (msg.type) {
        case 'get-preview':
            await getPreview();
            break;
        case 'convert-images':
            await convertImages();
            break;
        case 'export-frames':
            await exportFrames();
            break;
        case 'reset-images':
            await resetImages();
            break;
        case 'close-plugin':
            figma.closePlugin();
            break;
    }
};

// 미리보기 이미지 가져오기
async function getPreview() {
    try {
        console.log("🔄 미리보기 가져오기 시작");
        
        // simbol 프레임 찾기
        const simbolFrame = findFrameByName('simbol');
        if (!simbolFrame) {
            figma.ui.postMessage({ 
                type: 'preview-error', 
                message: 'simbol 프레임을 찾을 수 없습니다.' 
            });
            return;
        }
        
        // simbol 프레임 내 design 프레임 찾기
        const designFrame = findFrameByName('design', simbolFrame);
        if (!designFrame) {
            figma.ui.postMessage({ 
                type: 'preview-error', 
                message: 'simbol 프레임 내 design 프레임을 찾을 수 없습니다.' 
            });
            return;
        }
        
        // design 프레임을 이미지로 내보내기
        const imageBytes = await designFrame.exportAsync({
            format: 'PNG',
            constraint: { type: 'SCALE', value: 1 }
        });
        
        // base64로 인코딩
        const base64Data = figma.base64Encode(imageBytes);
        
        figma.ui.postMessage({
            type: 'preview-success',
            imageData: base64Data,
            frameName: designFrame.name
        });
        
        console.log("✅ 미리보기 가져오기 완료");
        
    } catch (error) {
        console.error("❌ 미리보기 오류:", error);
        figma.ui.postMessage({ 
            type: 'preview-error', 
            message: `미리보기 생성 중 오류가 발생했습니다: ${error.message}` 
        });
    }
}

// 이미지 변환 (복사/붙여넣기)
async function convertImages() {
    try {
        console.log("🔄 이미지 변환 시작");
        figma.ui.postMessage({ type: 'conversion-started' });
        
        // simbol 프레임 내 design 프레임 찾기
        const simbolFrame = findFrameByName('simbol');
        if (!simbolFrame) {
            figma.ui.postMessage({ 
                type: 'error', 
                message: 'simbol 프레임을 찾을 수 없습니다.' 
            });
            return;
        }
        
        const designFrame = findFrameByName('design', simbolFrame);
        if (!designFrame) {
            figma.ui.postMessage({ 
                type: 'error', 
                message: 'simbol 프레임 내 design 프레임을 찾을 수 없습니다.' 
            });
            return;
        }
        
        // design 프레임을 이미지로 내보내기
        const imageBytes = await designFrame.exportAsync({
            format: 'PNG',
            constraint: { type: 'SCALE', value: 1 }
        });
        
        // 이미지를 Figma에 추가
        const image = figma.createImage(imageBytes);
        
        // simbol, logo 프레임을 제외한 모든 프레임 찾기
        const targetFrames = figma.currentPage.findAll((node) => 
            node.type === 'FRAME' && 
            node.name !== 'simbol' && 
            node.name !== 'logo'
        );
        
        console.log("📋 대상 프레임 개수:", targetFrames.length);
        targetFrames.forEach(frame => console.log("  -", frame.name));
        
        if (targetFrames.length === 0) {
            figma.ui.postMessage({ 
                type: 'error', 
                message: '복사할 대상 프레임이 없습니다.' 
            });
            return;
        }
        
        // 각 프레임에 이미지 복사
        for (const frame of targetFrames) {
            console.log("📋 복사 중:", frame.name);
            await copyImageToFrame(image, frame);
        }
        
        console.log("✅ 이미지 변환 완료");
        figma.ui.postMessage({ type: 'conversion-completed' });
        
    } catch (error) {
        console.error("❌ 변환 오류:", error);
        figma.ui.postMessage({ 
            type: 'error', 
            message: `변환 중 오류가 발생했습니다: ${error.message}` 
        });
    }
}

// 프레임에 이미지 복사
async function copyImageToFrame(image, targetFrame) {
    try {
        // 기존 이미지 제거
        const existingImages = targetFrame.findAll((node) => 
            node.type === 'RECTANGLE' && 
            node.fills && 
            node.fills.some((fill) => fill.type === 'IMAGE')
        );
        existingImages.forEach((node) => node.remove());
        
        // 새 이미지 생성
        const newImage = figma.createRectangle();
        newImage.name = 'image';
        
        // 대상 프레임의 크기에 맞게 조정
        newImage.resize(targetFrame.width, targetFrame.height);
        
        // 이미지 채우기 설정
        newImage.fills = [{
            type: 'IMAGE',
            imageHash: image.hash,
            scaleMode: 'FILL'
        }];
        
        // 프레임에 추가
        targetFrame.appendChild(newImage);
        
        console.log(`✅ ${targetFrame.name}에 이미지 복사 완료`);
        
    } catch (error) {
        console.error(`❌ ${targetFrame.name} 복사 오류:`, error);
    }
}

// 프레임 내보내기 (선택)
async function exportFrames() {
    try {
        console.log("🔄 프레임 내보내기 시작");
        figma.ui.postMessage({ type: 'export-started' });
        
        // simbol, logo 프레임을 제외한 모든 프레임 찾기
        const targetFrames = figma.currentPage.findAll((node) => 
            node.type === 'FRAME' && 
            node.name !== 'simbol' && 
            node.name !== 'logo'
        );
        
        console.log("📋 내보낼 프레임 개수:", targetFrames.length);
        targetFrames.forEach(frame => console.log("  -", frame.name));
        
        if (targetFrames.length === 0) {
            figma.ui.postMessage({ 
                type: 'error', 
                message: '내보낼 프레임이 없습니다.' 
            });
            return;
        }
        
        // 각 프레임에 export 설정 추가
        for (const frame of targetFrames) {
            if (frame.exportSettings.length === 0) {
                frame.exportSettings = [{
                    format: 'PNG',
                    constraint: { type: 'SCALE', value: 1 }
                }];
            }
        }
        
        // 모든 프레임을 선택
        figma.currentPage.selection = targetFrames;
        figma.viewport.scrollAndZoomIntoView(targetFrames);
        
        console.log("✅ 프레임 내보내기 준비 완료");
        figma.ui.postMessage({ 
            type: 'export-completed',
            message: `${targetFrames.length}개의 프레임이 선택되었습니다. 우측 패널의 Export 버튼을 클릭하세요.`,
            frameCount: targetFrames.length
        });
        
    } catch (error) {
        console.error("❌ 내보내기 오류:", error);
        figma.ui.postMessage({ 
            type: 'error', 
            message: `내보내기 중 오류가 발생했습니다: ${error.message}` 
        });
    }
}

// 이미지 초기화 (삭제)
async function resetImages() {
    try {
        console.log("🔄 이미지 초기화 시작");
        figma.ui.postMessage({ type: 'reset-started' });
        
        // simbol, logo 프레임을 제외한 모든 프레임 찾기
        const targetFrames = figma.currentPage.findAll((node) => 
            node.type === 'FRAME' && 
            node.name !== 'simbol' && 
            node.name !== 'logo'
        );
        
        console.log("📋 초기화할 프레임 개수:", targetFrames.length);
        
        let deletedCount = 0;
        
        // 각 프레임에서 이미지 삭제
        for (const frame of targetFrames) {
            const imageNodes = frame.findAll((node) => 
                node.type === 'RECTANGLE' && 
                node.fills && 
                node.fills.some((fill) => fill.type === 'IMAGE')
            );
            
            imageNodes.forEach((node) => {
                node.remove();
                deletedCount++;
            });
            
            console.log(`🗑️ ${frame.name}에서 ${imageNodes.length}개 이미지 삭제`);
        }
        
        console.log("✅ 이미지 초기화 완료");
        figma.ui.postMessage({ 
            type: 'reset-completed',
            message: `${deletedCount}개의 이미지가 삭제되었습니다.`
        });
        
    } catch (error) {
        console.error("❌ 초기화 오류:", error);
        figma.ui.postMessage({ 
            type: 'error', 
            message: `초기화 중 오류가 발생했습니다: ${error.message}` 
        });
    }
}

// 이름으로 프레임 찾기
function findFrameByName(name, parent = null) {
    const searchArea = parent || figma.currentPage;
    return searchArea.findOne((node) => 
        node.type === 'FRAME' && 
        node.name === name
    );
}

console.log("🚀 Image Size Converter 플러그인 시작");
