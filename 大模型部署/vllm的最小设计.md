## block
bolck本身具有一个block_id, 用于唯一识别的ID, 为了多block共用以及缓存, 所以还需要一个hash, 一个ref_count, 前者用于计算是否cache hit, 后者用于实现是否这个block可以覆盖或者销毁, 或者说是返回到free_block_ids
## block manager
维护一个超大的blocks, 里面存放所有的block
维护一个free_block_ids, 用于提供空闲的block
维护一个used_block_ids,用于记录当前在使用的block

总而言之, 这个manager负责返回一个当前可用的block, 或者收回已经不用的block

