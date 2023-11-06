// Ben Mesnik

main = async () => {
    const vision = require('@google-cloud/vision');
    
    const client = new vision.ImageAnnotatorClient();

    const fileName = "calvinhobbespic3.jpg";
    const fileNameNoText = "calvinsledex2.png";
    const fileNameMultipleText = "calvinhobbes2captions8.jpg"; 

    console.log("DEBUG: Detecting text");

    // Performs text detection on the image file
    const [result] = await client.textDetection(fileName);
    if(result.textAnnotations.length == 0) {
        console.log("DEBUG: No text found");
        return JSON.stringify({
            "caption_1" : {},
            "caption_2" : {}
        }, null, 2);
    }
    const detections = result.textAnnotations;
    
    var captions = new Array(2);
    captions[0] = new Array();
    captions[1] = new Array();

    var captionLastPlacedWordIndexArr = new Array(2);
    var captionFurthestX = new Array(2);
    
    var caption1, caption1BottomRightX, caption1BottomRightY;
    const caption1Vertical = detections[1]["boundingPoly"]["vertices"][0]["x"];
    var caption1TopLeftX = caption1Vertical;
    var caption1TopLeftY = detections[1]["boundingPoly"]["vertices"][0]["y"];
    captionLastPlacedWordIndexArr[0] = 1;
    captionFurthestX[0] = detections[1]["boundingPoly"]["vertices"][2]["x"];

    var caption2, caption2Vertical, caption2TopLeftX, caption2TopLeftY, caption2BottomRightX, caption2BottomRightY;

    //collect total spacing between words for average
    var totalXSpacing = 0;
    var prevXCo = detections[1]["boundingPoly"]["vertices"][2]["x"];
    detections.slice(1).forEach(text => {
        spaceBeforeCurWord = text["boundingPoly"]["vertices"][0]["x"] - prevXCo;
        if(spaceBeforeCurWord > 0) {
            totalXSpacing += spaceBeforeCurWord;
        }
        prevXCo = text["boundingPoly"]["vertices"][2]["x"];
    });
    
    /////Multiple Caption Detection Settings//////
    
    const precision = 3;
    const newLineThreshold = 40;
    //this seems to work well for images of width 600
    const expectedWordDistanceThresholdSingleCaption = 8 * precision;
    
    //////////////////////////////////////////////

    const wordDistanceThreshold = (totalXSpacing / detections.length) * precision;
    
    console.log("DEBUG: Checking for multiple captions with" +
        "\n\t--Precision : " + precision + 
        "\n\t--Expected spacing threshold for a single caption : " + expectedWordDistanceThresholdSingleCaption + 
        "\n\n\t--Current spacing threshold : " + wordDistanceThreshold + "\n"
        );


    /* 
        if spacing greater than a specified average line spacing, probably multiple captions     
        
        (ex) threshold = 1
         ____________________________________________________
        |                                                    |
        | "example caption one"                              |
        |                              "example caption two" |
        |____________________________________________________|

                  ^       ^    ^^^^^^^^        ^       ^
                  1       1       8         1       1

                  avg = (1 + 1 + 8 + 1 + 1) / 6 = 2   ( > 1 )----> multiple
        
         ____________________________________________________
        |                                                    |
        |           "Lorem ipsum dolor sit                   |
        |            consectetur adipiscing"                 |
        |____________________________________________________|

                          ^     ^     ^
                          1     2     1

                  avg = (1 + 2 + 1) / 6 = 2/3         ( < 1 )----> single
    */
    if(wordDistanceThreshold > expectedWordDistanceThresholdSingleCaption) {
        console.log("DEBUG: Multiple captions detected");
        console.log("DEBUG: Full Text : " + detections[0]["description"].replace(new RegExp("\n", 'g'), "\\n"));
        var captionSelected = 0;
        
        //sort words into their proper caption
        //if spacing is regular continue to put word into selected caption bucket
        //if space is above the usual spacing, switch bucket to second caption
        //if space is negative (new line), identify which caption by matching it to the caption whos vertical is closest 
        detections.slice(1).forEach((text, i) => {
            curWordLeftXCo = text["boundingPoly"]["vertices"][0]["x"];
            spaceBeforeCurWord = curWordLeftXCo - prevXCo;
            if(spaceBeforeCurWord < 0) {
                if(typeof caption2Vertical == 'undefined' && Math.abs(caption1Vertical - curWordLeftXCo) > newLineThreshold) {
                    caption2Vertical = curWordLeftXCo;
                    caption2TopLeftX = text["boundingPoly"]["vertices"][0]["x"];
                    caption2TopLeftY = text["boundingPoly"]["vertices"][0]["y"];
                    captionFurthestX[1] = text["boundingPoly"]["vertices"][2]["x"];
                    captionSelected = 1;
                }
                else if(typeof caption2Vertical == 'undefined' || Math.abs(caption1Vertical - curWordLeftXCo) < Math.abs(caption2Vertical - curWordLeftXCo)) {
                    captionSelected = 0;
                } else {
                    captionSelected = 1;
                }
            }
            if(spaceBeforeCurWord > wordDistanceThreshold) {
                if(typeof caption2Vertical == 'undefined') {
                    caption2Vertical = curWordLeftXCo;
                    caption2TopLeftX = text["boundingPoly"]["vertices"][0]["x"];
                    caption2TopLeftY = text["boundingPoly"]["vertices"][0]["y"];
                    captionFurthestX[1] = text["boundingPoly"]["vertices"][2]["x"];
                }
                captionSelected = 1;
            } else if(spaceBeforeCurWord > wordDistanceThreshold / 2) {
                if(Math.abs(caption1Vertical - curWordLeftXCo) < Math.abs(caption2Vertical - curWordLeftXCo)) {
                    captionSelected = 0;
                } else {
                    if(typeof caption2Vertical == 'undefined') {
                        caption2Vertical = curWordLeftXCo;
                        caption2TopLeftX = text["boundingPoly"]["vertices"][0]["x"];
                        caption2TopLeftY = text["boundingPoly"]["vertices"][0]["y"];
                        captionFurthestX[1] = text["boundingPoly"]["vertices"][2]["x"];
                    }
                    captionSelected = 1;
                }
            }
            prevXCo = text["boundingPoly"]["vertices"][2]["x"];
            captionFurthestX[captionSelected] = Math.max(captionFurthestX[captionSelected], prevXCo);
            captionLastPlacedWordIndexArr[captionSelected] = i;      
            captions[captionSelected].push(text["description"]);
        });

        caption1BottomRightX = captionFurthestX[0];
        caption1BottomRightY = detections[captionLastPlacedWordIndexArr[0] + 1]["boundingPoly"]["vertices"][2]["y"];
        caption2BottomRightX = captionFurthestX[1];
        caption2BottomRightY = detections[captionLastPlacedWordIndexArr[1] + 1]["boundingPoly"]["vertices"][2]["y"];        

        caption1 = captions[0].join(" ").toLocaleLowerCase();
        caption2 = captions[1].join(" ").toLocaleLowerCase();

    } else {
        console.log("DEBUG: One caption found");
        caption1 = detections[0]["description"].replace(new RegExp("\n", 'g'), " ").toLocaleLowerCase();
        caption1BottomRightX = detections[0]["boundingPoly"]["vertices"][2]["x"];
        caption1BottomRightY = detections[0]["boundingPoly"]["vertices"][2]["y"];
    }

    console.log('RESULT:');

    const captionJSON = JSON.stringify({
        "caption_1" : {
            "text" : caption1,
            "top_left_x" : caption1TopLeftX,
            "top_left_y" : caption1TopLeftY,
            "bottom_right_x" : caption1BottomRightX,
            "bottom_right_y" : caption1BottomRightY
        },
        "caption_2" : {
            "text" : caption2,
            "top_left_x" : caption2TopLeftX,
            "top_left_y" : caption2TopLeftY,
            "bottom_right_x" : caption2BottomRightX,
            "bottom_right_y" : caption2BottomRightY
        }
    }, null, 2);

    console.log(captionJSON);
    
    return captionJSON;
}

main();
