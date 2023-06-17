# 基本命令

```postgresql
-- 启动postgresql
pg_ctl start

-- 进入数据库
psql database_name

-- 查看数据库表
\d
```



# debug步骤

1. 运行PGDev下的compile.sh

sh compile.sh

```bash
source ./env-debug

cd ./postgresql-12.5

make distclean

./configure --prefix=/home/suhan_2020152064/PGDev/target-debug \
        --enable-debug \
        CCFLAG="-g -O0" CC='gcc'

make -j 1 world

make install-world

cd ..

echo 'Run successful.'
```



2. 在tutorial目录下执行make clean，之后执行make install

3. 进入到~/PGDev/pghome/share/postgresql/contrib目录下
4. 修改sql目录的删除类型操作，G移动到文件末尾，dd删除一行

5. 创建数据库createdb testdb
6. 将sql文件导入db，psql -f complex.sql testdbcd 



7. 查看postgresql进程 ps -ef | grep postgres
8. 进入~/PGDev/pghome/bin文件下，使用gdb
9. 执行file postgres

![image-20230327220252104](C:\Users\ALIENWARE\AppData\Roaming\Typora\typora-user-images\image-20230327220252104.png)

10. attach （进程id），在步骤7中查看

11. 在postgres中执行操作语句
12. 在gdb下执行continue



加断点调试

1. 在.c文件中查看需要加断点的行数

![image-20230327220714737](C:\Users\ALIENWARE\AppData\Roaming\Typora\typora-user-images\image-20230327220714737.png)





# Join

实现连接的三种方法：

- 嵌套循环
- 排序合并
- 哈希



## 哈希连接

当join的谓词形式为：$T1.attr1 = T2.attr2$ 时，可以使用哈希连接进行运算，其中T1与T2分别为连接的两个关系，其中一个被指定为inner relation（T1），另一个被指定为outer relation（T2）



> 连接过程

常规哈希连接的阶段：

- 构建阶段
- 探测阶段



构建阶段：

对inner relation进行扫描，使用哈希函数f计算每个元组的连接属性的哈希值。随后元组插入到哈希表中



探测阶段：

扫描outer relation，寻找与inner relation匹配的元组。匹配过程分为两步，一、先使用相同的哈希函数f计算outer relation的连接属性的哈希值；二、如果哈希值对应桶不为空，匹配桶内的属性值，若真实值匹配成功









# 源码



<font style="background-color:yellow; color: red; font-size: 18px; font-weight: bold; float:left">结构体</font>

- HashJoinState：实现哈希连接算法的结构体，保存算法执行过程中的状态信息和中间结果

  - hj_JoinState：表示连接操作的状态
  - hj_HashTable：保存已经构建的哈希表
  - hj_CurBucketNo：当前处理的哈希桶编号，表示正在查找的桶的未知
  - hj_CurTuple：表示当前处理的元组的位置，是一个指向TupleTableSlot结构体的指针
  - hj_FirstOuterTupleSlot：指向第一个外部元组的指针
  - hj_NullOuterTupleSlot：外表（左表）空元组
  - hj_NullInnerTupleSlot：内表（右表）空元组

- HashState：实现散列表的基本方式
  - PlanState：用于存储执行计划节点的状态信息

  - HashJoinTable：标准是当前正在处理的散列表

  - hashkeys：散列表的key列表

- PlanState：pg中所有执行计划节点的状态结构体的基类

  - plan：指向执行计划节点的结构体

  - state：节点的状态，分为初始化状态、运行状态和结束状态

- ExprState：表示表达式的运算状态
  - resnull：表达式运算结果是否为null
  - resvalue：表达式运算结果的实际值
  - resultslot：表达式运算得到的结果的槽
  - expr：原始的表达式树

- ExprContext：表达式计算上下文中所有的状态信息
  - type：节点类型
  - ecxt_innertuple：内部表的当前行
  - ecxt_outertuple：外部表的当前行
- HashJoin：实现了实现哈希连接查询的任务
  - hashclauses：用来计算哈希值的字段列表（如：T1.id，T2.id）
  - hashoperators：执行哈希算法时用来计算哈希值的运算符列表
  - hashcollations：表示哈希算法使用的排序规则列表
  - hashkeys：表示哈希表中的键值



<font style="background-color:yellow; color: red; font-size: 18px; font-weight: bold; float:left">变量</font>

- node（HashJoinState）：执行计划
- outerNode（PlanState）：外表执行计划
- hashNode（HashState）：内表执行计划
- joinqual（ExprState）：连接操作（Join）应用的表达式，如：Join on
- otherqual（ExprState）：过滤操作中应用的表达式，如：Where
- econtext（ExprContext）：





<font style="background-color:yellow; color: red; font-size: 18px; font-weight: bold; float:left">方法</font>



```c
switch(hj_JoinState)
{
	case HJ_BUILD_HASHTABLE: 	// 构建hash表   
	case HJ_NEED_NEW_OUTER:  	// 从右侧取出元组
    case HJ_SCAN_BUCKET: 	 	// 扫描哈希桶
    case HJ_FILL_OUTER_TUPLE:	// 如果是左外连接，用null填充内表部分并输出
    case HJ_FILL_INNER_TUPLES: 	// 如果是右外连接，用null填充外表部分并输出
    case HJ_NEED_NEW_BATCH:		// 获取新的批次
}
```



```c
// 构造hashjoin节点，使得其左右子树均为hash执行计划
HashJoin *create_hashjoin_plan(PlannerInfo *root,
                              HashPath *best_path);
    
// 构造哈希表函数
// 混合哈希连接(MultiExecHash)：直接生成哈希表
// 对称哈希连接()：每获取一个元组，就往哈希表添加一个元组
MultiExecHash(HashState *node) --> ExecHash(HashState *node);
    
// 具体哈希连接实现
ExecHashJoinImpl(PlanState *pstate, bool parallel);

```



```c
static pg_attribute_always_inline TupleTableSlot *
ExecHashJoinImpl(PlanState *pstate, bool parallel)
{
	HashJoinState *node = castNode(HashJoinState, pstate);
	PlanState  *outerNode;
	HashState  *hashNode;
	ExprState  *joinqual;
	ExprState  *otherqual;
	ExprContext *econtext;
	HashJoinTable hashtable;
	TupleTableSlot *outerTupleSlot;
	uint32		hashvalue;
	int			batchno;
	ParallelHashJoinState *parallel_state;

	/*
	 * get information from HashJoin node
	 */
	joinqual = node->js.joinqual;
	otherqual = node->js.ps.qual;
	hashNode = (HashState *) innerPlanState(node);
	outerNode = outerPlanState(node);
	hashtable = node->hj_HashTable;
	econtext = node->js.ps.ps_ExprContext;
	parallel_state = hashNode->parallel_state;

	/*
	 * Reset per-tuple memory context to free any expression evaluation
	 * storage allocated in the previous tuple cycle.
	 */
	ResetExprContext(econtext);

	/*
	 * run the hash join state machine
	 */
	for (;;)
	{
		/*
		 * It's possible to iterate this loop many times before returning a
		 * tuple, in some pathological cases such as needing to move much of
		 * the current batch to a later batch.  So let's check for interrupts
		 * each time through.
		 */
		CHECK_FOR_INTERRUPTS();

		switch (node->hj_JoinState)
		{
			case HJ_BUILD_HASHTABLE:

				/*
				 * First time through: build hash table for inner relation.
				 */
				Assert(hashtable == NULL);

				/*
				 * If the outer relation is completely empty, and it's not
				 * right/full join, we can quit without building the hash
				 * table.  However, for an inner join it is only a win to
				 * check this when the outer relation's startup cost is less
				 * than the projected cost of building the hash table.
				 * Otherwise it's best to build the hash table first and see
				 * if the inner relation is empty.  (When it's a left join, we
				 * should always make this check, since we aren't going to be
				 * able to skip the join on the strength of an empty inner
				 * relation anyway.)
				 *
				 * If we are rescanning the join, we make use of information
				 * gained on the previous scan: don't bother to try the
				 * prefetch if the previous scan found the outer relation
				 * nonempty. This is not 100% reliable since with new
				 * parameters the outer relation might yield different
				 * results, but it's a good heuristic.
				 *
				 * The only way to make the check is to try to fetch a tuple
				 * from the outer plan node.  If we succeed, we have to stash
				 * it away for later consumption by ExecHashJoinOuterGetTuple.
				 */
				if (HJ_FILL_INNER(node))
				{
					/* no chance to not build the hash table */
					node->hj_FirstOuterTupleSlot = NULL;
				}
				else if (parallel)
				{
					/*
					 * The empty-outer optimization is not implemented for
					 * shared hash tables, because no one participant can
					 * determine that there are no outer tuples, and it's not
					 * yet clear that it's worth the synchronization overhead
					 * of reaching consensus to figure that out.  So we have
					 * to build the hash table.
					 */
					node->hj_FirstOuterTupleSlot = NULL;
				}
				else if (HJ_FILL_OUTER(node) ||
						 (outerNode->plan->startup_cost < hashNode->ps.plan->total_cost &&
						  !node->hj_OuterNotEmpty))
				{
					node->hj_FirstOuterTupleSlot = ExecProcNode(outerNode);
					if (TupIsNull(node->hj_FirstOuterTupleSlot))
					{
						node->hj_OuterNotEmpty = false;
						return NULL;
					}
					else
						node->hj_OuterNotEmpty = true;
				}
				else
					node->hj_FirstOuterTupleSlot = NULL;

				/*
				 * Create the hash table.  If using Parallel Hash, then
				 * whoever gets here first will create the hash table and any
				 * later arrivals will merely attach to it.
				 */
				hashtable = ExecHashTableCreate(hashNode,
												node->hj_HashOperators,
												node->hj_Collations,
												HJ_FILL_INNER(node));
				node->hj_HashTable = hashtable;

				/*
				 * Execute the Hash node, to build the hash table.  If using
				 * Parallel Hash, then we'll try to help hashing unless we
				 * arrived too late.
				 */
				hashNode->hashtable = hashtable;
				(void) MultiExecProcNode((PlanState *) hashNode);

				/*
				 * If the inner relation is completely empty, and we're not
				 * doing a left outer join, we can quit without scanning the
				 * outer relation.
				 */
				if (hashtable->totalTuples == 0 && !HJ_FILL_OUTER(node))
					return NULL;

				/*
				 * need to remember whether nbatch has increased since we
				 * began scanning the outer relation
				 */
				hashtable->nbatch_outstart = hashtable->nbatch;

				/*
				 * Reset OuterNotEmpty for scan.  (It's OK if we fetched a
				 * tuple above, because ExecHashJoinOuterGetTuple will
				 * immediately set it again.)
				 */
				node->hj_OuterNotEmpty = false;

				if (parallel)
				{
					Barrier    *build_barrier;

					build_barrier = &parallel_state->build_barrier;
					Assert(BarrierPhase(build_barrier) == PHJ_BUILD_HASHING_OUTER ||
						   BarrierPhase(build_barrier) == PHJ_BUILD_DONE);
					if (BarrierPhase(build_barrier) == PHJ_BUILD_HASHING_OUTER)
					{
						/*
						 * If multi-batch, we need to hash the outer relation
						 * up front.
						 */
						if (hashtable->nbatch > 1)
							ExecParallelHashJoinPartitionOuter(node);
						BarrierArriveAndWait(build_barrier,
											 WAIT_EVENT_HASH_BUILD_HASHING_OUTER);
					}
					Assert(BarrierPhase(build_barrier) == PHJ_BUILD_DONE);

					/* Each backend should now select a batch to work on. */
					hashtable->curbatch = -1;
					node->hj_JoinState = HJ_NEED_NEW_BATCH;

					continue;
				}
				else
					node->hj_JoinState = HJ_NEED_NEW_OUTER;

				/* FALL THRU */

			case HJ_NEED_NEW_OUTER:

				/*
				 * We don't have an outer tuple, try to get the next one
				 */
				if (parallel)
					outerTupleSlot =
						ExecParallelHashJoinOuterGetTuple(outerNode, node,
														  &hashvalue);
				else
					outerTupleSlot =
						ExecHashJoinOuterGetTuple(outerNode, node, &hashvalue);

				if (TupIsNull(outerTupleSlot))
				{
					/* end of batch, or maybe whole join */
					if (HJ_FILL_INNER(node))
					{
						/* set up to scan for unmatched inner tuples */
						ExecPrepHashTableForUnmatched(node);
						node->hj_JoinState = HJ_FILL_INNER_TUPLES;
					}
					else
						node->hj_JoinState = HJ_NEED_NEW_BATCH;
					continue;
				}

				econtext->ecxt_outertuple = outerTupleSlot;
				node->hj_MatchedOuter = false;

				/*
				 * Find the corresponding bucket for this tuple in the main
				 * hash table or skew hash table.
				 */
				node->hj_CurHashValue = hashvalue;
				ExecHashGetBucketAndBatch(hashtable, hashvalue,
										  &node->hj_CurBucketNo, &batchno);
				node->hj_CurSkewBucketNo = ExecHashGetSkewBucket(hashtable,
																 hashvalue);
				node->hj_CurTuple = NULL;

				/*
				 * The tuple might not belong to the current batch (where
				 * "current batch" includes the skew buckets if any).
				 */
				if (batchno != hashtable->curbatch &&
					node->hj_CurSkewBucketNo == INVALID_SKEW_BUCKET_NO)
				{
					bool		shouldFree;
					MinimalTuple mintuple = ExecFetchSlotMinimalTuple(outerTupleSlot,
																	  &shouldFree);

					/*
					 * Need to postpone this outer tuple to a later batch.
					 * Save it in the corresponding outer-batch file.
					 */
					Assert(parallel_state == NULL);
					Assert(batchno > hashtable->curbatch);
					ExecHashJoinSaveTuple(mintuple, hashvalue,
										  &hashtable->outerBatchFile[batchno]);

					if (shouldFree)
						heap_free_minimal_tuple(mintuple);

					/* Loop around, staying in HJ_NEED_NEW_OUTER state */
					continue;
				}

				/* OK, let's scan the bucket for matches */
				node->hj_JoinState = HJ_SCAN_BUCKET;

				/* FALL THRU */

			case HJ_SCAN_BUCKET:

				/*
				 * Scan the selected hash bucket for matches to current outer
				 */
				if (parallel)
				{
					if (!ExecParallelScanHashBucket(node, econtext))
					{
						/* out of matches; check for possible outer-join fill */
						node->hj_JoinState = HJ_FILL_OUTER_TUPLE;
						continue;
					}
				}
				else
				{
					if (!ExecScanHashBucket(node, econtext))
					{
						/* out of matches; check for possible outer-join fill */
						node->hj_JoinState = HJ_FILL_OUTER_TUPLE;
						continue;
					}
				}

				/*
				 * We've got a match, but still need to test non-hashed quals.
				 * ExecScanHashBucket already set up all the state needed to
				 * call ExecQual.
				 *
				 * If we pass the qual, then save state for next call and have
				 * ExecProject form the projection, store it in the tuple
				 * table, and return the slot.
				 *
				 * Only the joinquals determine tuple match status, but all
				 * quals must pass to actually return the tuple.
				 */
				if (joinqual == NULL || ExecQual(joinqual, econtext))
				{
					node->hj_MatchedOuter = true;

					if (parallel)
					{
						/*
						 * Full/right outer joins are currently not supported
						 * for parallel joins, so we don't need to set the
						 * match bit.  Experiments show that it's worth
						 * avoiding the shared memory traffic on large
						 * systems.
						 */
						Assert(!HJ_FILL_INNER(node));
					}
					else
					{
						/*
						 * This is really only needed if HJ_FILL_INNER(node),
						 * but we'll avoid the branch and just set it always.
						 */
						HeapTupleHeaderSetMatch(HJTUPLE_MINTUPLE(node->hj_CurTuple));
					}

					/* In an antijoin, we never return a matched tuple */
					if (node->js.jointype == JOIN_ANTI)
					{
						node->hj_JoinState = HJ_NEED_NEW_OUTER;
						continue;
					}

					/*
					 * If we only need to join to the first matching inner
					 * tuple, then consider returning this one, but after that
					 * continue with next outer tuple.
					 */
					if (node->js.single_match)
						node->hj_JoinState = HJ_NEED_NEW_OUTER;

					if (otherqual == NULL || ExecQual(otherqual, econtext))
						return ExecProject(node->js.ps.ps_ProjInfo);
					else
						InstrCountFiltered2(node, 1);
				}
				else
					InstrCountFiltered1(node, 1);
				break;

			case HJ_FILL_OUTER_TUPLE:

				/*
				 * The current outer tuple has run out of matches, so check
				 * whether to emit a dummy outer-join tuple.  Whether we emit
				 * one or not, the next state is NEED_NEW_OUTER.
				 */
				node->hj_JoinState = HJ_NEED_NEW_OUTER;

				if (!node->hj_MatchedOuter &&
					HJ_FILL_OUTER(node))
				{
					/*
					 * Generate a fake join tuple with nulls for the inner
					 * tuple, and return it if it passes the non-join quals.
					 */
					econtext->ecxt_innertuple = node->hj_NullInnerTupleSlot;

					if (otherqual == NULL || ExecQual(otherqual, econtext))
						return ExecProject(node->js.ps.ps_ProjInfo);
					else
						InstrCountFiltered2(node, 1);
				}
				break;

			case HJ_FILL_INNER_TUPLES:

				/*
				 * We have finished a batch, but we are doing right/full join,
				 * so any unmatched inner tuples in the hashtable have to be
				 * emitted before we continue to the next batch.
				 */
				if (!ExecScanHashTableForUnmatched(node, econtext))
				{
					/* no more unmatched tuples */
					node->hj_JoinState = HJ_NEED_NEW_BATCH;
					continue;
				}

				/*
				 * Generate a fake join tuple with nulls for the outer tuple,
				 * and return it if it passes the non-join quals.
				 */
				econtext->ecxt_outertuple = node->hj_NullOuterTupleSlot;

				if (otherqual == NULL || ExecQual(otherqual, econtext))
					return ExecProject(node->js.ps.ps_ProjInfo);
				else
					InstrCountFiltered2(node, 1);
				break;

			case HJ_NEED_NEW_BATCH:

				/*
				 * Try to advance to next batch.  Done if there are no more.
				 */
				if (parallel)
				{
					if (!ExecParallelHashJoinNewBatch(node))
						return NULL;	/* end of parallel-aware join */
				}
				else
				{
					if (!ExecHashJoinNewBatch(node))
						return NULL;	/* end of parallel-oblivious join */
				}
				node->hj_JoinState = HJ_NEED_NEW_OUTER;
				break;

			default:
				elog(ERROR, "unrecognized hashjoin state: %d",
					 (int) node->hj_JoinState);
		}
	}
}
```



```c
static TupleTableSlot *
ExecParallelHashJoinOuterGetTuple(PlanState *outerNode,
								  HashJoinState *hjstate,
								  uint32 *hashvalue)
{
	HashJoinTable hashtable = hjstate->hj_HashTable;
	int			curbatch = hashtable->curbatch;
	TupleTableSlot *slot;

	/*
	 * In the Parallel Hash case we only run the outer plan directly for
	 * single-batch hash joins.  Otherwise we have to go to batch files, even
	 * for batch 0.
	 */
	if (curbatch == 0 && hashtable->nbatch == 1)
	{
		slot = ExecProcNode(outerNode);

		while (!TupIsNull(slot))
		{
			ExprContext *econtext = hjstate->js.ps.ps_ExprContext;

			econtext->ecxt_outertuple = slot;
			if (ExecHashGetHashValue(hashtable, econtext,
									 hjstate->hj_OuterHashKeys,
									 true,	/* outer tuple */
									 HJ_FILL_OUTER(hjstate),
									 hashvalue))
				return slot;

			/*
			 * That tuple couldn't match because of a NULL, so discard it and
			 * continue with the next one.
			 */
			slot = ExecProcNode(outerNode);
		}
	}
	else if (curbatch < hashtable->nbatch)
	{
		MinimalTuple tuple;

		tuple = sts_parallel_scan_next(hashtable->batches[curbatch].outer_tuples,
									   hashvalue);
		if (tuple != NULL)
		{
			ExecForceStoreMinimalTuple(tuple,
									   hjstate->hj_OuterTupleSlot,
									   false);
			slot = hjstate->hj_OuterTupleSlot;
			return slot;
		}
		else
			ExecClearTuple(hjstate->hj_OuterTupleSlot);
	}

	/* End of this batch */
	return NULL;
}
```



```c

static void
MultiExecPrivateHash(HashState *node)
{
	PlanState  *outerNode;
	List	   *hashkeys;
	HashJoinTable hashtable;
	TupleTableSlot *slot;
	ExprContext *econtext;
	uint32		hashvalue;

	/*
	 * get state info from node
	 */
	outerNode = outerPlanState(node);
	hashtable = node->hashtable;

	/*
	 * set expression context
	 */
	hashkeys = node->hashkeys;
	econtext = node->ps.ps_ExprContext;

	/*
	 * Get all tuples from the node below the Hash node and insert into the
	 * hash table (or temp files).
	 */
	for (;;)
	{
		slot = ExecProcNode(outerNode);
		if (TupIsNull(slot))
			break;
		/* We have to compute the hash value */
		econtext->ecxt_outertuple = slot;
		if (ExecHashGetHashValue(hashtable, econtext, hashkeys,
								 false, hashtable->keepNulls,
								 &hashvalue))
		{
			int			bucketNumber;

			bucketNumber = ExecHashGetSkewBucket(hashtable, hashvalue);
			if (bucketNumber != INVALID_SKEW_BUCKET_NO)
			{
				/* It's a skew tuple, so put it into that hash table */
				ExecHashSkewTableInsert(hashtable, slot, hashvalue,
										bucketNumber);
				hashtable->skewTuples += 1;
			}
			else
			{
				/* Not subject to skew optimization, so insert normally */
				ExecHashTableInsert(hashtable, slot, hashvalue);
			}
			hashtable->totalTuples += 1;
		}
	}

	/* resize the hash table if needed (NTUP_PER_BUCKET exceeded) */
	if (hashtable->nbuckets != hashtable->nbuckets_optimal)
		ExecHashIncreaseNumBuckets(hashtable);

	/* Account for the buckets in spaceUsed (reported in EXPLAIN ANALYZE) */
	hashtable->spaceUsed += hashtable->nbuckets * sizeof(HashJoinTuple);
	if (hashtable->spaceUsed > hashtable->spacePeak)
		hashtable->spacePeak = hashtable->spaceUsed;

	hashtable->partialTuples = hashtable->totalTuples;
}
```



- `src/backend/executor/nodeHash.c`  	                创建哈希表		
- `src/backend/executor/nodeHashjoin.c`             进行哈希连接
- `src/backend/optimizer/plan/createplan.c`     创建计划节点
- `src/include/executor/nodeHash.h`                      哈希函数声明
- `src/include/nodes/execnodes.h`                          数据结构修改（HashJoinState结构）





# 对称哈希连接

[![img](https://p3-passport.byteimg.com/img/mosaic-legacy/3792/5112637127~100x100.awebp)](https://juejin.cn/user/298716944678702)

[旺旺来啦![创作等级LV.2](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/53901be7d27e4abe94530e12d815491d~tplv-k3u1fbpfcp-no-mark:0:0:0:0.awebp)](https://juejin.cn/user/298716944678702)

2022年05月08日 20:28 · 阅读 796

关注

![改造pg的hashjoin形式为Symmetric Hash Join](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd65703b02ee41799629c6c0910d02e6~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?)

## 一：背景

最近在做数据库内核原理的实验二的作业的时候一开始看pg代码遇到勒巨大的困难。所以在完成该作业之后希望能够将写作业的时候的经验记录下来，希望能给后面的人给带来一点作业启示。

### 1.1 要求如下：

将pg中使用到的hybrid hashjoin算法切换为Symmetric Hash Join。 ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb74a010e1fc403fb5057409b628729e~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 1.2 实验环境：

- 系统环境：ubuntu18.04
- pg版本：postgresql-12.5
- 修改后的代码仓库：[pg-Symmetric-Hash-Join(github.com)](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fjomeswang%2Fpg-Symmetric-Hash-Join)

## 二：从hash算法（hybrid_hash和symmetric_hash Join）形式了解pg的hash过程

对于pg处理hash过程而言分为两个阶段：

- **构建阶段：** 构建hash表
- **探查阶段：** 每次都去探查构建出来的hash表看有没有满足条件的行数据

### 2.1 构建角度：

我们如果想要将pg自带的hybrid_hashjoin修改为Symmetric Hash Join。 就需要先了解Symmetric Hash Join算法是如何处理inner数据和outer数据进行适配的。
**这是hybrid hashjoin的构建图** ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d998112fa7924452b38b2d11f261e477~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?) **这是Symmetric Hash Join的构建图** ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/070d6b45cde2473cba4f146517dceff9~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)
可以发现对于hybrid hashjoin而言它只会为inner_table构建一个hash_table而对于Symmetric Hash Join而言它会为inner_table和outer_table分别构建一个hash_table，所以最后会有两个hash_table。

### 2.2 探查搜索角度

**hybrid_hash_join的探查方式** 对于hybrid_hash_join而言它是采用先构建inner_hash表之后再去用outer_tuple去探查inner_hash表中是否有满足条件的数据。

**symmetric_hash_join的探查方式** 而对于symmetric_hash_join它是采用边构建边探查的形式去寻找匹配的数据的。 流程如下：

- 从内部关系中读取一个元组，并使用内部关系的哈希函数将其插入到内部关系的哈希表中。然后，使用新元组来探测外部关系的哈希表是否匹配。探测时使用外部关系的哈希函数。
- 当用内部元组探测没有更多匹配时，从外部关系中读取一个元组。使用外部关系的哈希函数将其插入到外部关系的哈希表中。然后，使用外部元组来探测内部关系的哈希表是否匹配，计算哈希值时使用内部表的哈希函数。
- `持续执行上述两个操作`。直到当外部关系读取完之后，继续一直读取内部关系。同理当内部关系读取完成之后继续一直读取外部关系

## 三：梳理改造点

### 3.1 修改配置项

1. 要求测试的时候只使用hashjoin的形式而不使用nest_loop或者merge_join的形式，所以我们需要在 `postgresql.conf`这一个文件中配置一下关闭这两种join方法。 ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/938601b964b14df4b8329c89efbc2d94~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)
2. 关闭多batch模式 由于在hybrid_hash_join模式中，是使用多个批次分批来处理元组的，但是在本实验中可以假设两个hash表都适合内存。 所以我们需要修改这一个多batch模式。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1f9fc603d884b7d9c53e8b7cca59099~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

### 3.2 修改实现逻辑项

#### 3.2.1 修改 createplan.c

该文件主要做的事情就是把`路径树转换为计划树`，并且整理筛选一些数据段。 之前只是对innerplan做了转换，这次要增加对 outerplan也进行转换。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ddbb11eda5ea4c31a271855707925b71~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)



#### 3.2.2 修改 execnodes.h

文件包含 HashJoinState 结构，该结构用于执行过程中维护哈希连接的状态。 修改这一个文件，源文件中只对inner的状态进行了管理，这次要增加对outer的状态管理和outer_hash_table的管理。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65b5426d2f5542719de81b481e21ca51~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 3.2.3 修改 nodeHash.c

该文件主要是为了`构建与维护哈希表`
之前该文件的并没有实现流水线的哈希表构建，所以我们要在 ExecHash 中进行补充实现。 ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3cad6f7489c41b49c92f73b03913fab~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?) 再而我们需要修改之前探查哈希表找匹配值的代码：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/79691ccfa8a9497fb5a530d5c324a83b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

这次我们需要增加一下从内部表找匹配值和从外部表中找匹配值得代码

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7815854d6c44941935c13634c7d1883~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 3.2.4 修改 nodeHash.h

文件包含 nodeHash.c 的函数原型。 在这里我们增加一下 ExecScanHashBucket_probeouter， ExecScanHashBucket_probeinner的原型（扫描外表，扫描内表） ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e4773c61a9849e6a2d20624a2bcf79d~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

#### 3.2.5 修改 nodeHashjoin.c

该文件主要负责实现哈希连接运算，判断什么值需要连接返回
在这里需要修改三个函数

- 在`ExecHashJoinImpl函数`中进行节点连接返回（判断连接）
- 在`ExecInitHashJoin函数`中进行连接状态初始化操作
- 在`ExecEndHashJoin函数`中进行连接状态销毁操作 原实现是在`ExecHashJoinImpl函数`中进行连接返回的。 在该函数中使用状态机进行状态管理来执行。 ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/12184999ff954dee84a9be527637274c~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?) 于是在这里我们需要去修改它的状态机机制去做连接判断。 ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d105675af13f410cb66ccadc8d549e62~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?) 具体实现可看 [Comparing 8ddadc3..9efe41f · jomeswang/pg-Symmetric-Hash-Join (github.com)](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fjomeswang%2Fpg-Symmetric-Hash-Join%2Fcompare%2F8ddadc3..9efe41f)

`ExecInitHashJoin函数`中主要是添加了一些状态的初始化 ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3691c0883de140d4b2a95f3240a805d8~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

`ExecInitHashJoin函数`中主要是添加了一些状态的销毁

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/29913e0195ce42f991c8ed48ff48bf94~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

## 总结





















