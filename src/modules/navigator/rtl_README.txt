进入RTL_MODE的流程说明：
一，将_rtl_start_lock = false， RTL_MODE模式中，CLIMB/TURN/RETURN模式中，都pos_sp_triplet中的previous_setpoint
      和current_setpoint都是valid的，next_setpoint 都是invalid的。DESEND/LAND模式中,只有current_setpoint是valid，
      为了防止漂移（航迹飞行程序决定control_auto）,以上这些setpoint的设置与_rtl_start_lock相关。
二，将当前位置信息赋值给_mission_item;
三，将_mission_item赋值给pos_sp_triplet->current;
       current_setpoint航迹点的水平速度这里赋值，MPC_XY_CRUISE。
四，计算home_dist,当前位置与home position的水平距离。
五，1，如果此时是landed，会直接进入RTL_STATE_LANDED;
        2，如果此时是takeoff，但离home position的距离小于5m，或者当前高度大于home_position->alt + RTL_RETURN_ALT，不会爬升，而会直接进入RTL_STATE_TURN;
        3，如果此时是takeoff，离home position 的距离大于5m,并且当前高度小于home_position->alt + RTL_RETURN_ALT，会进入RTL_STATE_CLIMB;
 六，_set_rtl_item()函数的作用：
 	1，根据当前的_rtl_state来更改_mission_item,然后将_mission_item赋值给pos_sp_triplet->current，实际上是设置当前的航迹点；
 	2，同时根据当前的_rtl_state来决定pos_sp_triplet->previous.valid 是否为true，这会影响control_auto的具体执行步骤。
 	3，将pos_sp_triplet->next.valid = false，这同样影响control_auto的具体执行步骤。
 七，现在代码中，RTL_STATE_TURN是我后来人为加上的，这样是为了使返航时更加平稳一些，
         可以对头返航，也可以对尾返航。我倾向于对尾返航，因为这样对航向的调整相对较小，更平稳。
         但是如果用于航拍的话，还是对头返回效果更好。
 八，为了使返航时更加平稳，更改了几个参数：
 	MPC_XY_CRUISE == 3.0f, 这个参数的更改会影响mission中的水平飞行速度，原来是5.0f，感觉太快，后期应该可以在地面站中进行设置。
 	MPC_Z_VEL_MAX_UP == 1.0f, 爬升速度，这个参数同样会影响mission的垂直速度，原来是3.0f。
 	MPC_LAND_SPEED == 0.2f, 这个降落速度比较理想，不会出现降落不稳的现象。
 九，RTL中有两个高度设定的参数：
 	RTL_RETURN_ALT：这个参数是返航高度，当大于这个高度时，维持当前高度返航，小于这个高度时，爬升。
 	RTL_DESCEND_ALT：这个参数是开始降落的高度，当到达这个高度时，飞机进入RTL_STATE_LAND。
 	注意区分这两个参数。


RTL_MODE的流程一般是：
CLIMB——TURN——RETURN——DESCED——LAND——LANDED