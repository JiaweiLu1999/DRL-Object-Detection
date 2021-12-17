# 12.17 Patch

## Modifications
- change `compute_reward` in agent.py:
```
def compute_reward(self, actual_state, previous_state, ground_truth):
    /.../
    if res <= 0:
        return -1
->  return 0 
```

- change `train` in agent.py:

```
def train(self, train_loader, verbose = False):
    /.../
->  # shuffle gt_boxes for generalization
    random.shuffle(ground_truth_boxes)
    
    # iterate gt_box to learn cross
->  for ground_truth in ground_truth_boxes:
        # initialization setting
        self.actions_history = torch.zeros((9,self.n_actions))
        new_image = image

    /.../ 
```
- change `predict_image` in agent.py

```
def predict_image(self, image, plot=False, verbose=False,
                original_bdbox = [0,224,0,224],
                bdboxes = [], maskboxes = []):

    /.../

    # start interaction
    while not done:
        steps += 1
        # take action according to greedy policy
        action = self.select_action_model(state)

->      if action == 0:
            if self.is_repeated_trigger(self.current_coord, bdboxes):
                max_gt = self.get_max_bdbox(bdboxes, self.current_coord)
                gt_index = bdboxes.index(max_gt)
                mask, new_mask_box = self.create_mask(maskboxes[gt_index], bdboxes[gt_index])
            else:
                mid_point_x = round((self.current_coord[1] + self.current_coord[0])/2)
                mid_point_y = round((self.current_coord[3] + self.current_coord[2])/2)
                mid_point = [mid_point_x, mid_point_x, mid_point_y, mid_point_y]
                mask, new_mask_box = self.create_mask(mid_point, self.current_coord)
                gt_index = -1
            next_state = None
            new_equivalent_coord = self.current_coord
            done = True
        else:
            /.../

        if steps == 20:
->          mask, new_mask_box, gt_index = None, None, None
            done = True
->          cross_flag = False

        state = next_state
        image = new_image
    
    /.../

    return new_equivalent_coord, cross_flag, steps, mask, new_mask_box, gt_index
        
```

- change `predict_multiple_objects` in agent.py

```
def predict_multiple_objects(self, image, plot=False, verbose=False):

    /.../
    bdboxes = []
    maskboxes = []

    while 1:
        bdbox, cross_flag, steps, mask, new_mask_box, gt_index = self.predict_image(new_image, plot, verbose, original_bdboxes[i%5], bdboxes, maskboxes)

        if cross_flag:
            new_image *= mask
            if gt_index == -1:
                bdboxes.append(bdbox)
                maskboxes.append(new_mask_box)
            elif gt_index >= 0:
                maskboxes[gt_index] = new_mask_box
    
    /.../
    return bdboxes

```

## Result

### after change value function:
-  trianing stat
    - Class cat...
        Predicting boxes...
        Computing recall and ap...
        Final result : 
                      0.5
        ap      67.780749
        recall  83.246073


    - Class cow...
        Predicting boxes...
        Computing recall and ap...
        Final result : 
                      0.5
        ap      12.770240
        recall  34.054054

    - Class dog...
        Predicting boxes...
        Computing recall and ap...
        Final result : 
                      0.5
        ap      32.915763
        recall  72.693727

    - Class bird...
        Predicting boxes...
        Computing recall and ap...
        Final result : 
                      0.5
        ap      47.466922
        recall  57.482993

    - Class car...
        Predicting boxes...
        Computing recall and ap...
        Final result : 
                      0.5
        ap      18.634854
        recall  39.830508
        
- Validation stat

    - Class cat...
        Predicting boxes...
        Computing recall and ap...
        Final result : 
                      0.5
        ap      42.745037
        recall  46.969697
        
    - Class cow...
        Predicting boxes...
        Computing recall and ap...
        Final result : 
                      0.5
        ap      10.246107
        recall  13.450292


    - Class dog...
        Predicting boxes...
        Computing recall and ap...
        Final result : 
                      0.5
        ap      32.632685
        recall  41.947566
        
    - Class bird...
        Predicting boxes...
        Computing recall and ap...
        Final result : 
                      0.5
        ap      17.508418
        recall  16.393443
        
    - Class car...
        Predicting boxes...
        Computing recall and ap...
        Final result : 
                      0.5
        ap      15.541472
        recall  23.105134

### after change prediction:

- Validation stat

    - Class cat...
        Predicting boxes...
        Computing recall and ap...
        Final result : 
                      0.5
        ap      52.144622
        recall  50.000000

    - Class cow...
        Predicting boxes...
        Computing recall and ap...
        Final result : 
                      0.5
        ap      17.272727
        recall  16.959064
        
    - Class dog...
        Predicting boxes...
        Computing recall and ap...
        Final result : 
                      0.5
        ap      43.547755
        recall  44.943820
        
    - Class bird...
        Predicting boxes...
        Computing recall and ap...
        Final result : 
                      0.5
        ap      25.038107
        recall  20.327869
        
    - Class car...
        Predicting boxes...
        Computing recall and ap...
        Final result : 
                      0.5
        ap      25.828571
        recall  26.039120