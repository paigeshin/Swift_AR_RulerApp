# Swift_AR_RulerApp

### Node라는 개념과 Material 개념이 중요한듯. 

- node는 화면에 존재하는 object들을 의미
- material은 꾸며줌
- 보통 패턴이 SCN 시리즈로 불리우는 미리 만들어진 object를 material로 꾸며주고 node에 추가함. 


```Swift


//
//  ViewController.swift
//  AR_Ruler
//
//  Created by shin seunghyun on 2020/05/02.
//  Copyright © 2020 shin seunghyun. All rights reserved.
//

import UIKit
import SceneKit
import ARKit

class ViewController: UIViewController, ARSCNViewDelegate {

    @IBOutlet var sceneView: ARSCNView!
    
    var dotNodes: [SCNNode] = [SCNNode]()
    var textNode = SCNNode()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Set the view's delegate
        sceneView.delegate = self
        
        //add Debug Option
        sceneView.debugOptions = [ARSCNDebugOptions.showFeaturePoints]
        
    }
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        
        // Create a session configuration
        let configuration = ARWorldTrackingConfiguration()

        // Run the view's session
        sceneView.session.run(configuration)
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        
        // Pause the view's session
        sceneView.session.pause()
    }
    
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        super.touchesBegan(touches, with: event)
        
        //dot이 이미 2개 이상이면 지워줌. 재사용 가능하게 만들기 위해서임.
        if dotNodes.count >= 2 {
            for dot in dotNodes {
                dot.removeFromParentNode()
            }
            dotNodes = [SCNNode]()
        }
        
        if let touchLocation = touches.first?.location(in: sceneView) {
            let hitTestResults = sceneView.hitTest(touchLocation, types: .featurePoint) //array를 반납 - 각각의 array는 먼저 발견한 object를 reference하는 값을 가지고 있다.
            if let hitResult = hitTestResults.first {
                addDot(at: hitResult) //유저가 touch한 부분에 그림을 그려줌
            }
        }
        
    }
    
    //In order to specify location in the real world
    func addDot(at hitResult: ARHitTestResult) {
        
        let dotGeometry = SCNSphere(radius: 0.005)
        let material = SCNMaterial()
        material.diffuse.contents = UIColor.red
        dotGeometry.materials = [material]
        let dotNode = SCNNode(geometry: dotGeometry)
        dotNode.position = SCNVector3(
            hitResult.worldTransform.columns.3.x,
            hitResult.worldTransform.columns.3.y,
            hitResult.worldTransform.columns.3.z
        )
        sceneView.scene.rootNode.addChildNode(dotNode)
        
        dotNodes.append(dotNode) //어레이에 결과값을 저장.
        
        if dotNodes.count >= 2 {
            calculate()
        }

    }

    //실제로 두 점 사이의 거리를 구해주는 함수
    //sqrt -> 루트
    //pow -> exponential
    //abs -> 절대 값
    func calculate() {
        
        let start = dotNodes[0]
        let end = dotNodes[1]
        
        //high school math
        let a = end.position.x - start.position.x
        let b = end.position.y - start.position.y
        let c = end.position.z - start.position.z
        
        let distance = sqrt(
                pow(a, 2) +
                pow(b, 2) +
                pow(c, 2)
            )
        
        updateText(text: "\(abs(distance))", atPosition: end.position)

    }
    
    //distance 결과 값을 보여주는 text를 출력
    func updateText(text: String, atPosition position: SCNVector3) {
        
        textNode.removeFromParentNode()
        
        let textGeometry = SCNText(string: text, extrusionDepth: 1.0)
        textGeometry.firstMaterial?.diffuse.contents = UIColor.red
        textNode = SCNNode(geometry: textGeometry)
        textNode.position = SCNVector3(position.x, position.y + 0.01, position.z)
        textNode.scale = SCNVector3(x: 0.01, y: 0.01, z: 0.01)
        sceneView.scene.rootNode.addChildNode(textNode)
    }
    

    
}
```
