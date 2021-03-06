Note to people other than myself: I uploaded this code from an apple sample project so I could study it remotely

Command Organization and Execution Model:
https://developer.apple.com/library/ios/documentation/Miscellaneous/Conceptual/MetalProgrammingGuide/Cmd-Submiss/Cmd-Submiss.html#//apple_ref/doc/uid/TP40014221-CH3-SW1
	

Render command encoder - 1 Draw Call
Buffer (MTLBuffer)
	- Vertex buffer
Texture (MTLTexture)

Sampler state
Depth stencil state (MTLDepthStencilState)
Render pipeline state (MTLRenderPipelineState)
	- Vertex shader
	- Fragment shader
	- Pixel formats

Command buffer - 1 FRAME (
contains a frame's worth of render command encoder calls

Command (buffer) queue - Buffered Frames (MTLCommandQueue)


Object relationships
Metal layer (CAMetalLayer)
	• Drawable (CAMetalDrawable)
		Render pass descriptor (MTLRenderPassDescriptor) - depends on a drawable
			.colorAttachments[0] (texture, loadAction, clearColor, storeAction)
			.depthAttachments (texture, loadAction, clearColor, storeAction)

	• Device (MTLDevice)
		- Command queue (MTLCommandQueue)
			- Command buffer (MTLCommandBuffer)
				- Command encoder (MTLRenderCommandEncoder)
				- Completed handler

		- Vertex and uniform buffer (MTLBuffer)

		- Depth and stencil state (MTLDepthStencilState)

		- Library (MTLLibrary)
			- Vertex program (MTLFunction)
			- Fragment program (MTLFunction)


Encode rendering commands for a drawable (a framebuffer)
// Wait for the semaphore to be set
SemaphoreWait(mySemaphore)

// New command buffer
- Get a CommandBuffer from the CommandQueue

// Create a render command encoder with a render pass descriptor
- Get a MetalDrawable (framebuffer) from the MetalLayer
- Create a RenderPassDescriptor with MetalDrawable.texture (the framebuffer)
- Using the CommandBuffer, create a RenderCommandEncoder with the RenderPassDescriptor

// Set render command encoder state
- RenderCommandEncoder:
	.RenderPipelineState = myPipelineState (vert, frag, pixel format)
	.DepthStencilState = myDepthStencilState (z-buffer state)
	.VertexBuffer[0] = myVertexBuffer
	.VertexBuffer[1] = myUniformsBuffer (model view projection matrix)

// Draw
- RenderCommandEncoder.DrawPrimitives(Triangles)

// Submit
- RenderCommandEncoder.EndEncoding()

// Schedule to clear the wait semaphore
- CommandBuffer.AddCompletedHandler( SemaphoreSignal(mySemaphore) ) // Clear the wait semaphore on completion
- (uniform buffer index++) % numberOfIndexes // could this not be in the callback? (no, b/c it happens for the next draw)

- CommandBuffer.PresentDrawable(myDrawable) // Schedule to show the framebuffer
- RenderCommandEncoder.Commit() // Finish

