---
title: 'Scanning codes in iOS'
media_order: uinavigation.png
date: '19:45 10/31/2020'
taxonomy:
    category:
        - blog
    tag:
        - iOS
        - AVFoundation
theme: magnet-theme
feed:
    limit: 10
hero_classes: 'text-dark title-h1h2 overlay-light hero-large parallax'
hero_image: unsplash-luca-bravo.jpg
blog_url: /blog
show_sidebar: false
show_breadcrumbs: true
show_pagination: true
subtitle: 'finding beauty in structure'
---

The AVFoundation framework offers a pretty good way to scan codes with the device camera, such as qr codes or barcodes.
This blog post gives an implementation of doing such code scanning.

===

Subsequently is the implementation of this feature. It also offers the possibility to toggle torch light on and off to facility scanning.
After the view is loaded a captureSession is obtained and video input is locked.

If some code is recognized, AVFoundations tells us so by calling the delegate methods from AVCaptureMetadataOutputObjectsDelegate.

The metadataObjectTypes property of AVCaptureMetadataOutput allows us to filter for different codes, such as qr codes or bar codes.

```
final class ScanCodeViewController: UIViewController, AVCaptureMetadataOutputObjectsDelegate {

override func viewDidLoad() {
        super.viewDidLoad()
        
        scan()
    }

override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)

        if captureSession?.isRunning == false {
            captureSession?.startRunning()
        }
    }

    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)

        if captureSession?.isRunning == true {
            captureSession?.stopRunning()
        }
    }

private func setupNavigation() {
        if let device = AVCaptureDevice.default(for: .video), device.hasTorch {
            createTorchBarItem(with: Asset.Images.flashOn.image)
        }
    }
    
    private func createTorchBarItem(with image: ImageAsset.Image) {
        navigationItem.leftBarButtonItem = UIBarButtonItem.buttonItem(
            image: image,
            target: self,
            action: #selector(toggleTorch))
    }

private func scan() {
        captureSession = AVCaptureSession()

        guard let videoCaptureDevice = AVCaptureDevice.default(for: .video), let captureSession = captureSession else {
            logWarning("Could not retrieve capture session", tag: Self.tag)
            return
        }
        
        let videoInput: AVCaptureDeviceInput

        do {
            videoInput = try AVCaptureDeviceInput(device: videoCaptureDevice)
        } catch {
            logWarning("Could not capture video input", tag: Self.tag)
            return
        }
        
        if (captureSession.canAddInput(videoInput)) {
            captureSession.addInput(videoInput)
        } else {
            failed()
            return
        }

        let metadataOutput = AVCaptureMetadataOutput()

        if (captureSession.canAddOutput(metadataOutput)) {
            captureSession.addOutput(metadataOutput)

            metadataOutput.setMetadataObjectsDelegate(self, queue: DispatchQueue.main)
            metadataOutput.metadataObjectTypes = [.code128]
        } else {
            failed()
            return
        }

        let previewLayer = AVCaptureVideoPreviewLayer(session: captureSession)
        previewLayer.frame = view.layer.bounds
        previewLayer.videoGravity = .resizeAspectFill
        view.layer.addSublayer(previewLayer)
        
        self.previewLayer = previewLayer

        captureSession.startRunning()
    }
    
    private func failed() {
        let alertController = UIAlertController(title: L10n.scanningNotSupported, message: L10n.scanningNotSupportedAlertMessage, preferredStyle: .alert)
        alertController.addAction(UIAlertAction(title: L10n.ok, style: .default))
        present(alertController, animated: true)
        captureSession = nil
    }
    
    func metadataOutput(_ output: AVCaptureMetadataOutput, didOutput metadataObjects: [AVMetadataObject], from connection: AVCaptureConnection) {
        captureSession?.stopRunning()

        if let metadataObject = metadataObjects.first {
            guard let readableObject = metadataObject as? AVMetadataMachineReadableCodeObject else { return }
            guard let stringValue = readableObject.stringValue else { return }
            AudioServicesPlaySystemSound(SystemSoundID(kSystemSoundID_Vibrate))
            found(code: stringValue)
        }

        dismiss(animated: true)
    }
    
    func found(code: String) {
        logInfo("Scanned barcode \(code)", tag: Self.tag)
        
        delegate?.didScan(code: code)
    }
    
    @objc func toggleTorch() {
        guard let device = AVCaptureDevice.default(for: .video) else {
            logWarning("Could not capture video input for torch", tag: Self.tag)
            return
        }
        
        do {
            try device.lockForConfiguration()

            if !device.isTorchActive {
                device.torchMode = .on
                createTorchBarItem(with: Asset.Images.flashOff.image)
            } else {
                device.torchMode = .off
                createTorchBarItem(with: Asset.Images.flashOn.image)
            }
            
            device.unlockForConfiguration()
        } catch {
            logWarning("Torch could not be used", tag: Self.tag)
        }
    }
}
```

