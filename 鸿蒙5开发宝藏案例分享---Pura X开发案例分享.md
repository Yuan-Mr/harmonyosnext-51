### 🌟 HarmonyOS Treasure Case Sharing: Pura X Outer Screen Development Battle Analysis  

Hello everyone! I'm your HarmonyOS development partner. Today, while flipping through official documentation, I accidentally discovered Huawei's hidden "treasure-level" case—the Pura X foldable outer screen development practice! These practical solutions are incredibly useful, yet surprisingly little discussed. I quickly organized them into essential tips to share with you. It's full of insights—remember to save it!  


### 🧩 I. Why Special Adaptation for the Outer Screen Is Needed?  
Pura X adopts a top-bottom folding design, with the outer screen being a 1:1 square screen (inner screen 16:10). This means:  
● The outer screen height is only 1/2 of the inner screen (approximately 24vp vs 48vp)  
● Operation methods must be single-hand friendly (高频 operations like checking notifications, payments, navigation)  
● Special handling of layout compression/content truncation issues  

The official team provides solutions for 5 core scenarios, which we'll拆解 step by step below 👇  


### 🚀 II. Development Practice for Five Scenarios (with Code Analysis)  
#### 1️⃣ Small-Window Responsive Layout  
**Pain point**: The same component requires different sizes on inner/outer screens.  
**Solution**: Dynamically adjust styles via dual breakpoint judgment.  

```typescript
Image($r('app.media.icon'))
.height(
  // Key judgment: landscape breakpoint sm + portrait breakpoint md
  this.currentWidthBreakpoint === 'sm' && 
  this.currentHeightBreakpoint === 'md' 
    ? 24 // Outer screen height
    : 48 // Inner screen height
)
.aspectRatio(1) // Maintain square shape
```  

#### 2️⃣ Content Visibility Control  
**Pain point**: The outer screen needs to hide non-core elements (e.g., Banner ads).  
**Solution A**: Control display with `visibility`.  

```typescript
Column() {
  Text("Outer screen exclusive content").fontSize(12)
}
.visibility(
  this.currentWidthBreakpoint === 'sm' && 
  this.currentHeightBreakpoint === 'md' 
    ? Visibility.Visible 
    : Visibility.None
)
```  

**Solution B**: Conditional rendering (suitable for complex components).  

```typescript
if (this.isSmallScreen) { // Custom judgment function
  Column() {
    Button("Outer screen quick payment")
  }
}
```  

#### 3️⃣ Sliding Container Optimization  
**Pain point**: Insufficient outer screen height causes content truncation.  
**God operation**: Use `Scroll` component + automatic invalidation mechanism.  

```typescript
Scroll() {
  Column() { /* Main content */ }
}
.scrollBar(BarState.Off) // Hide scrollbar
.height('100%')
```  
**Highlight**: When content height < container height, `Scroll` automatically disables sliding without manual judgment!  

#### 4️⃣ Immersive Adaptation for Short Videos  
**Effect**: Full-screen immersive video + semi-transparent floating controls.  

```typescript
Stack() {
  // 1. Bottom-layer video (full screen)
  Image($r('app.media.video_bg'))
    .objectFit(ImageFit.Cover) // Key: Maintain proportion filling

  // 2. Top title bar (avoid notch)
  Row() { ... }
    .padding({ top: $r('app.float.topAvoidHeight') }) 

  // 3. Side floating controls (with elastic blank space)
  Scroll() {
    Column() {
      Blank().layoutWeight(3) // Upper elastic placeholder
      Button("Like") 
      Blank().layoutWeight(1) // Lower elastic placeholder
    }
  }
}
```  

#### 5️⃣ Sliding Hidden Controls (Ultra-Practical!)  
**Interaction effect**:  
● Swipe up → Hide title bar/bottom bar  
● Swipe down → Gradually restore display  
**Core code**:  

```typescript
// Listen to scroll offset
.onScrollFrameBegin((offset: number) => {
  if (offset > 0) { // Swipe up
    this.topBarHeight *= 0.95 // Gradually reduce height
    this.barOpacity -= 0.05  // Gradually reduce opacity
  } else { // Swipe down
    animateTo({ duration: 300 }, () => {
      this.topBarHeight = 78 + avoidHeight // Restore height
      this.barOpacity = 1                  // Restore opacity
    })
  }
})
```  


### 💡 III. Pitfall Prevention Guide  
#### 1. System Avoidance Area Handling  
Must use `getWindowAvoidArea()` to obtain notch/chin height:  
```typescript
const topAvoid = window.getWindowAvoidArea(AvoidAreaType.TYPE_SYSTEM)
AppStorage.setOrCreate('topAvoidHeight', topAvoid.topRect.height)
```  

#### 2. Fold Status Listening  
```typescript
// Listen to fold status changes
foldStatus: FoldStatus = FoldStatus.FOLD_STATUS_EXPANDED

aboutToAppear() {
  foldController.on('foldStatusChange', (status) => {
    this.foldStatus = status
  })
}
```  

#### 3. Transition Animation Optimization  
Ensure smoothness from inner to outer screen with page continuation:  
```typescript
// Define shared elements in pageTransition
PageTransition() {
  PageTransitionEnter({ duration: 250 })
    .sharedTransition('sharedButton')
}
```  


### 🔚 IV. Conclusion  
The Pura X outer screen adaptation solution unearthed this time is简直 a "golden manual" for foldable development! Especially the designs for sliding hidden controls and dual-breakpoint responsiveness can be directly applied to other HarmonyOS device development.  

Finally, a word for you: The foldable ecosystem is booming. Mastering these technologies now will make you the brightest candidate for a salary increase next year! 💪  

(Original collation is not easy—if you find it useful, please like to let me know! The next issue will share the "HarmonyOS distributed camera development" battle practice!)
