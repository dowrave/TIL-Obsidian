
```python
tf.keras.losses.BinaryFocalCrossentropy(apply_class_balancing=False,   
										alpha=0.25,    
										gamma=2.0,    
										from_logits=False,    
										label_smoothing=0.0,    
										axis=-1,    
										reduction=losses_utils.ReductionV2.AUTO,    
										name='binary_focal_crossentropy')
```
- `model.compile(loss = )` 에 전달하면 됨.
- 