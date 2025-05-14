# qwen2.5 填充权重

填充权重

```python
# https://github.com/QwenLM/Qwen2.5/issues/578
def padding_and_saving_weight(model: torch.nn.Module, output_dir: str):
    assert model.config.intermediate_size == 29568, "intermediate_size 不是 29568"
    
    pad_size = 128
    old_intermediate_size=29568
    new_intermediate_size=old_intermediate_size+pad_size

    exponent = math.log(pad_size, 2)
    assert exponent.is_integer(), f"{pad_size} 不是2的次方数"
    exponent = int(exponent)

    assert (old_intermediate_size/pad_size).is_integer(), f"{old_intermediate_size} 不能被 {pad_size} 整除"

    need_pad_values = [int(old_intermediate_size // (2 ** i)) for i in range(exponent + 1)]

    sd = model.state_dict()
    for i, k in enumerate(sd):
        v = sd[k]
        if len(v.shape) == 2 and ( ('mlp.up_proj.' in k) or ('mlp.gate_proj.' in k) or ('mlp.down_proj.' in k)):
            if v.shape[0] in need_pad_values :
                need_pad_size = v.shape[0]*new_intermediate_size/old_intermediate_size - v.shape[0]
                assert need_pad_size.is_integer() , f"{need_pad_size} 不是整数"
                need_pad_size = int(need_pad_size)
                
                prev_v = F.pad(v.unsqueeze(1), (0, 0, 0, 1, 0, 0)).reshape(v.shape[0]*2, -1)[:need_pad_size*2]
                new_v = torch.cat([prev_v, v[need_pad_size:]], dim=0)
                sd[k] = new_v
                print(k, i, v.shape, '-->', new_v.shape)
            elif   v.shape[1] in need_pad_values:
                need_pad_size = v.shape[1]*new_intermediate_size/old_intermediate_size - v.shape[1]
                assert need_pad_size.is_integer() , f"{need_pad_size} 不是整数"
                need_pad_size = int(need_pad_size)
                
                prev_v= F.pad(v.unsqueeze(2), (0, 1)).reshape(v.shape[0], v.shape[1]*2)[:, :need_pad_size*2]
                new_v = torch.cat([prev_v, v[:, need_pad_size:]], dim=1)
                sd[k] = new_v
                print(k, i, v.shape, '-->', new_v.shape)
                
    model.config.intermediate_size=new_intermediate_size
    model.save_pretrained(output_dir, state_dict=sd, max_shard_size="4GB", safe_serialization=True)
    
    
```
