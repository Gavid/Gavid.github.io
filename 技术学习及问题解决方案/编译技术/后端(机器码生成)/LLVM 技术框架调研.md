https://getting-started-with-llvm-core-libraries-zh-cn.readthedocs.io/zh_CN/latest/ch07.html#llvm-jit



* 引入包

  ```java
  dependencies {
  //  api 'org.bytedeco:javacpp:1.5.7'
  //  api 'org.bytedeco:llvm:9.0.0-1.5.2'
    // https://mvnrepository.com/artifact/org.bytedeco/llvm-platform
  //  implementation group: 'org.bytedeco', name: 'llvm-platform', version: '12.0.1-1.5.6'
  //  implementation group: 'org.bytedeco', name: 'llvm', version: '12.0.1-1.5.6'
    implementation group: 'org.bytedeco', name: 'javacpp-platform', version: '1.5.8'
    implementation group: 'org.bytedeco', name: 'javacpp', version: '1.5.8'
    implementation 'org.bytedeco:llvm:15.0.3-1.5.8:macosx-arm64'
    implementation 'org.bytedeco:llvm:15.0.3-1.5.8'
  }
  ```

* 代码调试逻辑

```java
public void getLLVM(SClass root) {
  // Stage 1: Initialize LLVM components
  LLVMInitializeCore(LLVMGetGlobalPassRegistry());
  LLVMInitializeNativeAsmPrinter();
  LLVMInitializeNativeAsmParser();
  LLVMInitializeNativeDisassembler();
  LLVMInitializeNativeTarget();
  // Stage 2: Build the sum function
  LLVMOrcThreadSafeContextRef threadContext = LLVMOrcCreateNewThreadSafeContext();
  LLVMContextRef context = LLVMOrcThreadSafeContextGetContext(threadContext);
  LLVMModuleRef module = LLVMModuleCreateWithNameInContext(root.getLocation().getSourceName(), context);
  LLVMBuilderRef builder = LLVMCreateBuilderInContext(context);
  LLVMTypeRef i32Type = LLVMInt32TypeInContext(context);

  Map<String, LLVMValueRef> parameters = new HashMap<>();
  Map<String, LLVMTypeRef> parameterTypes = new HashMap<>();

  //for (int index = 0; index < root.getFunctionNodes().size(); index++) {
  final SFunction functionNode = root.getFunctionNodes().get(0);
  LLVMValueRef functionValueRef;
  // 定义函数及其对应的参数配置
  final int paramsSize = functionNode.getCanonicalTypeNameParameters().size();
  PointerPointer<Pointer> argumentTypes = new PointerPointer<>(paramsSize);
  for (int i = 0; i < paramsSize; i++) {
    argumentTypes.put(i, i32Type);
  }
  functionValueRef = LLVMAddFunction(module, functionNode.getFunctionName(), LLVMFunctionType(i32Type, argumentTypes, paramsSize, 0)); // LLVMGetTypeByName2(context, functionNode.getReturnCanonicalTypeName())
  LLVMSetFunctionCallConv(functionValueRef, LLVMCCallConv);
  LLVMSetLinkage(functionValueRef, LLVMExternalLinkage);
  final List<String> parameterNames = functionNode.getParameterNames();

  for (int i = 0; i < parameterNames.size(); i++) {
    parameters.put(parameterNames.get(i), LLVMGetParam(functionValueRef, i));
    parameterTypes.put(parameterNames.get(i), i32Type);
  }

  // 定义函数体
  // 创建一个基本块
  LLVMBasicBlockRef entryBlock = LLVMAppendBasicBlockInContext(context, functionValueRef, "entry");

  // 在基本块中添加指令
  LLVMPositionBuilderAtEnd(builder, entryBlock);

  for (AStatement statementNode : functionNode.getBlockNode().getStatementNodes()) {
    if (statementNode instanceof SDeclBlock) {
      final List<SDeclaration> declarationNodes = ((SDeclBlock) statementNode).getDeclarationNodes();
      for (SDeclaration sDeclaration : declarationNodes) {
        final AExpression valueNode1 = sDeclaration.getValueNode();
        if (valueNode1 instanceof ENewArray) {
          ENewArray eNewArray = (ENewArray)valueNode1;
          LLVMTypeRef int_array_type = LLVMArrayType(i32Type, eNewArray.getValueNodes().size());
          LLVMValueRef my_array = LLVMBuildAlloca(builder, int_array_type, sDeclaration.getSymbol());

          if (eNewArray.isInitializer()) {
            for (int i = 0; i < eNewArray.getValueNodes().size(); i++) {
              final AExpression valueNode = eNewArray.getValueNodes().get(i);
              if (valueNode instanceof ENumeric) {
                ENumeric eNumeric = (ENumeric) valueNode;
                PointerPointer<Pointer> indices = new PointerPointer<>(1);
                indices.put(0, LLVMConstInt(i32Type, 0, 0));
                indices.put(1, LLVMConstInt(i32Type, i, 0));
                LLVMValueRef arrayElement = LLVMBuildGEP2(builder, int_array_type, my_array, indices, 2, "myArrayElement");
                LLVMBuildStore(builder, LLVMConstInt(i32Type, Long.parseLong(eNumeric.getNumeric()), 0), arrayElement);
              }
            }
          } else {
            LLVMBuildStore(builder, LLVMConstNull(int_array_type), my_array);
          }
          parameters.put(sDeclaration.getSymbol(), my_array);
          parameterTypes.put(sDeclaration.getSymbol(), int_array_type);
          continue;
        }
        if (valueNode1 instanceof EUnary) {
          EUnary eUnary = (EUnary)valueNode1;
          final Operation operation = eUnary.getOperation();
          final ENumeric childNode = (ENumeric)eUnary.getChildNode();

          LLVMValueRef valueRef1 = LLVMBuildAlloca(builder, i32Type, sDeclaration.getSymbol());

          LLVMValueRef valueRef = null;
          if (operation == Operation.SUB) {
            valueRef = LLVMBuildSub(builder, LLVMConstInt(i32Type, 0, 0), LLVMConstInt(i32Type, Long.parseLong(childNode.getNumeric()), 0), "-1");
          }

          LLVMBuildStore(builder, valueRef, valueRef1);
          parameters.put(sDeclaration.getSymbol(), valueRef1);
          parameterTypes.put(sDeclaration.getSymbol(), i32Type);
          continue;
        }
        if (valueNode1 instanceof EBinary) {
          EBinary eBinary = (EBinary)valueNode1;
          final LLVMValueRef valueRef2 = LLVMBuildAlloca(builder, i32Type, sDeclaration.getSymbol());
          final ESymbol eSymbol = (ESymbol)eBinary.getLeftNode();
          final Operation operation = eBinary.getOperation();
          final ENumeric eNumeric = (ENumeric)eBinary.getRightNode();

          LLVMValueRef valueRef3 = null;
          if (operation == Operation.REM){
            valueRef3 = LLVMBuildSRem(builder, parameters.get(eSymbol.getSymbol()), LLVMConstInt(i32Type, Integer.parseInt(eNumeric.getNumeric(), eNumeric.getRadix()), 0), "x%10");
          }
          LLVMBuildStore(builder, valueRef3, valueRef2);
          parameters.put(sDeclaration.getSymbol(), valueRef3);
          parameterTypes.put(sDeclaration.getSymbol(), i32Type);
          continue;
        }
      }
    }

    if (statementNode instanceof SIfElse) {
      SIfElse sIfElse = (SIfElse)statementNode;
      final EComp conditionNode = (EComp)sIfElse.getConditionNode();
      final ESymbol leftNode = (ESymbol)conditionNode.getLeftNode();
      final LLVMValueRef leftNodeValueRef = parameters.get(leftNode.getSymbol());

      final EBrace rightNode = (EBrace)conditionNode.getRightNode();
      final ESymbol prefixNode = (ESymbol)rightNode.getPrefixNode();
      final ENumeric indexNode = (ENumeric)rightNode.getIndexNode();
      PointerPointer<Pointer> indices = new PointerPointer<>(1);
      indices.put(0, LLVMConstInt(i32Type, Integer.parseInt(indexNode.getNumeric(), indexNode.getRadix()), 0));
      LLVMValueRef arrayElement = LLVMBuildGEP2(builder, parameterTypes.get(prefixNode.getSymbol()), parameters.get(prefixNode.getSymbol()), indices, 1, "myArrayElement");
      final LLVMValueRef rightNodeValueRef = LLVMBuildLoad2(builder, i32Type, arrayElement, prefixNode.getSymbol() + "[" + indexNode.getNumeric() + "]");
      final Operation operation = conditionNode.getOperation();
      LLVMValueRef cmp = null;
      if (operation == Operation.EQ) {
        cmp = LLVMBuildICmp(builder, LLVMIntEQ, leftNodeValueRef, rightNodeValueRef, "cmp");
      }

      final SBlock ifBlockNode = sIfElse.getIfBlockNode();
      final SBlock elseBlockNode = sIfElse.getElseBlockNode();
      LLVMBasicBlockRef ifBlock = LLVMAppendBasicBlock(functionValueRef, "ifBlock");
      LLVMBasicBlockRef elseBlock = LLVMAppendBasicBlock(functionValueRef, "elseBlock");
      LLVMBasicBlockRef endBlock = LLVMAppendBasicBlock(functionValueRef, "endBlock");

      LLVMBuildCondBr(builder, cmp, ifBlock, elseBlock);

      LLVMPositionBuilderAtEnd(builder, ifBlock);
      for (AStatement node : ifBlockNode.getStatementNodes()) {
        if(node instanceof SExpression) {
          SExpression sExpression = (SExpression)node;
          final EAssignment eAssignment = (EAssignment)sExpression.getStatementNode();
          final ESymbol leftNode1 = (ESymbol) eAssignment.getLeftNode();
          final LLVMValueRef leftValueRef = parameters.get(leftNode1.getSymbol());

          final EBrace rightNode1 = (EBrace)eAssignment.getRightNode();
          final ENumeric indexNode1 = (ENumeric)rightNode1.getIndexNode();
          final ESymbol prefixNode1 = (ESymbol)rightNode1.getPrefixNode();
          PointerPointer<Pointer> indices1 = new PointerPointer<>(1);
          indices1.put(0, LLVMConstInt(i32Type, Integer.parseInt(indexNode1.getNumeric(), indexNode1.getRadix()), 0));
          LLVMValueRef arrayElement1 = LLVMBuildGEP2(builder, parameterTypes.get(prefixNode1.getSymbol()), parameters.get(prefixNode1.getSymbol()), indices1, 1, "myArrayElement");
          final LLVMValueRef rightValueRef = LLVMBuildLoad2(builder, i32Type, arrayElement1, prefixNode1.getSymbol() + "[" + indexNode1.getNumeric() + "]");

          LLVMBuildStore(builder, rightValueRef, leftValueRef);
        }
      }
      LLVMBuildBr(builder, endBlock);

      LLVMPositionBuilderAtEnd(builder, elseBlock);
      // 初始化 -1
      final LLVMValueRef valueRef2 = LLVMBuildAlloca(builder, i32Type, "ret");

      final LLVMValueRef valueRef12 = LLVMBuildSub(builder, LLVMConstInt(i32Type, 0, 0), LLVMConstInt(i32Type, 1, 0), "-1");

      LLVMBuildStore(builder, valueRef12, valueRef2);
      for (AStatement node : elseBlockNode.getStatementNodes()) {
        if(node instanceof SIfElse) {
          SIfElse sIfElse1 = (SIfElse)node;
          final EBooleanComp conditionNode1 = (EBooleanComp)sIfElse1.getConditionNode();
          // 条件1
          final EComp leftNode2 = (EComp)conditionNode1.getLeftNode();
          final ESymbol leftNode1 = (ESymbol)leftNode2.getLeftNode();
          final LLVMValueRef valueRef = parameters.get(leftNode1.getSymbol());

          final EBrace rightNode2 = (EBrace)leftNode2.getRightNode();
          final ENumeric indexNode1 = (ENumeric)rightNode2.getIndexNode();
          final ESymbol prefixNode1 = (ESymbol)rightNode2.getPrefixNode();
          PointerPointer<Pointer> indices1 = new PointerPointer<>(1);
          indices1.put(0, LLVMConstInt(i32Type, Integer.parseInt(indexNode1.getNumeric(), indexNode1.getRadix()), 0));
          LLVMValueRef arrayElement1 = LLVMBuildGEP2(builder, parameterTypes.get(prefixNode1.getSymbol()), parameters.get(prefixNode1.getSymbol()), indices1, 1, "myArrayElement");
          final LLVMValueRef rightValueRef = LLVMBuildLoad2(builder, i32Type, arrayElement1, prefixNode1.getSymbol() + "[" + indexNode1.getNumeric() + "]");
          final Operation operation1 = leftNode2.getOperation();
          LLVMValueRef cond1 = null;
          if (operation1 == Operation.GT) {
            cond1 = LLVMBuildICmp(builder, LLVMIntSGT, valueRef, rightValueRef, "cond1");
          }
          // 条件2
          final EComp rightNode1 = (EComp)conditionNode1.getRightNode();
          final ESymbol leftNodeTemp = (ESymbol)rightNode1.getLeftNode();
          final LLVMValueRef valueRef1 = parameters.get(leftNodeTemp.getSymbol());

          final EBrace rightNode3 = (EBrace)rightNode1.getRightNode();
          final ENumeric indexNode2 = (ENumeric)rightNode3.getIndexNode();
          final ESymbol prefixNode2 = (ESymbol)rightNode3.getPrefixNode();
          PointerPointer<Pointer> indices2 = new PointerPointer<>(1);
          indices2.put(0, LLVMConstInt(i32Type, Integer.parseInt(indexNode2.getNumeric(), indexNode2.getRadix()), 0));
          LLVMValueRef arrayElement2 = LLVMBuildGEP2(builder, parameterTypes.get(prefixNode2.getSymbol()), parameters.get(prefixNode2.getSymbol()), indices2, 1, "myArrayElement");
          final LLVMValueRef rightValueRef1 = LLVMBuildLoad2(builder, i32Type, arrayElement2, prefixNode2.getSymbol() + "[" + indexNode2.getNumeric() + "]");
          final Operation operation2 = rightNode1.getOperation();
          LLVMValueRef cond2 = null;
          if (operation2 == Operation.LTE) {
            cond2 = LLVMBuildICmp(builder, LLVMIntSLE, valueRef1, rightValueRef1, "cond2");
          }
          // 条件1 和条件2 比较
          final Operation operation3 = conditionNode1.getOperation();

          LLVMValueRef cmpCommon = null;
          if (operation3 == Operation.AND) {
            cmpCommon = LLVMBuildAnd(builder, cond1, cond2, "cmp");
          }

          final SBlock ifBlockNode1 = sIfElse1.getIfBlockNode();
          final SBlock elseBlockNode1 = sIfElse1.getElseBlockNode();
          LLVMBasicBlockRef ifBlock1 = LLVMAppendBasicBlock(functionValueRef, "ifBlock1");
          LLVMBasicBlockRef elseBlock1 = LLVMAppendBasicBlock(functionValueRef, "elseBlock1");
          LLVMBasicBlockRef endBlock1 = LLVMAppendBasicBlock(functionValueRef, "endBlock1");

          LLVMBuildCondBr(builder, cmpCommon, ifBlock1, elseBlock1);

          LLVMPositionBuilderAtEnd(builder, ifBlock1);
          for (AStatement node1 : ifBlockNode1.getStatementNodes()) {
            if(node1 instanceof SExpression) {
              SExpression sExpression1 = (SExpression)node1;
              final EAssignment eAssignment1 = (EAssignment)sExpression1.getStatementNode();
              final ESymbol leftNode10 = (ESymbol) eAssignment1.getLeftNode();
              final LLVMValueRef leftValueRef10 = parameters.get(leftNode10.getSymbol());

              final EBrace rightNode10 = (EBrace)eAssignment1.getRightNode();
              final ENumeric indexNode10 = (ENumeric)rightNode10.getIndexNode();
              final ESymbol prefixNode10 = (ESymbol)rightNode10.getPrefixNode();
              PointerPointer<Pointer> indices10 = new PointerPointer<>(1);
              indices10.put(0, LLVMConstInt(i32Type, Integer.parseInt(indexNode10.getNumeric(), indexNode10.getRadix()), 0));
              LLVMValueRef arrayElement10 = LLVMBuildGEP2(builder, parameterTypes.get(prefixNode10.getSymbol()), parameters.get(prefixNode10.getSymbol()), indices10, 1, "myArrayElement");
              final LLVMValueRef rightValueRef10 = LLVMBuildLoad2(builder, i32Type, arrayElement10, prefixNode10.getSymbol() + "[" + indexNode10.getNumeric() + "]");

              LLVMBuildStore(builder, rightValueRef10, leftValueRef10);
            }
          }
          LLVMBuildBr(builder, endBlock1);

          LLVMPositionBuilderAtEnd(builder, elseBlock1);
          for (AStatement node1 : elseBlockNode1.getStatementNodes()) {
            if(node1 instanceof SExpression) {
              SExpression sExpression1 = (SExpression)node1;
              final EAssignment eAssignment1 = (EAssignment)sExpression1.getStatementNode();
              final ESymbol leftNode10 = (ESymbol) eAssignment1.getLeftNode();
              final LLVMValueRef leftValueRef10 = parameters.get(leftNode10.getSymbol());

              final EBrace rightNode10 = (EBrace)eAssignment1.getRightNode();
              final ENumeric indexNode10 = (ENumeric)rightNode10.getIndexNode();
              final ESymbol prefixNode10 = (ESymbol)rightNode10.getPrefixNode();
              PointerPointer<Pointer> indices10 = new PointerPointer<>(1);
              indices10.put(0, LLVMConstInt(i32Type, Integer.parseInt(indexNode10.getNumeric(), indexNode10.getRadix()), 0));
              LLVMValueRef arrayElement10 = LLVMBuildGEP2(builder, parameterTypes.get(prefixNode10.getSymbol()), parameters.get(prefixNode10.getSymbol()), indices10, 1, "myArrayElement");
              final LLVMValueRef rightValueRef10 = LLVMBuildLoad2(builder, i32Type, arrayElement10, prefixNode10.getSymbol() + "[" + indexNode10.getNumeric() + "]");

              LLVMBuildStore(builder, rightValueRef10, leftValueRef10);
            }
          }
          LLVMBuildBr(builder, endBlock1);

          LLVMPositionBuilderAtEnd(builder, endBlock1);

        }
      }
      LLVMBuildBr(builder, endBlock);

      LLVMPositionBuilderAtEnd(builder, endBlock);
    }

    if (statementNode instanceof SReturn) {
      SReturn sReturn = (SReturn)statementNode;
      final ESymbol valueNode = (ESymbol)sReturn.getValueNode();
      LLVMBuildRet(builder, LLVMBuildLoad2(builder, parameterTypes.get(valueNode.getSymbol()),parameters.get(valueNode.getSymbol()), "ret"));
    }
  }

  //}


  // 主函数
  final SFunction functionNode1 = root.getFunctionNodes().get(1);
  LLVMValueRef functionValueRef1;
  // 定义函数及其对应的参数配置
  final int paramsSize1 = functionNode1.getCanonicalTypeNameParameters().size();
  try (PointerPointer<Pointer> argumentTypes1 = new PointerPointer<>(paramsSize1)){
    for (int i = 0; i < paramsSize1; i++) {
      argumentTypes.put(i, i32Type);
    }
    functionValueRef1 = LLVMAddFunction(module, functionNode1.getFunctionName(), LLVMFunctionType(i32Type, argumentTypes1, paramsSize1, 0)); // LLVMGetTypeByName2(context, functionNode.getReturnCanonicalTypeName())
  }
  LLVMSetFunctionCallConv(functionValueRef1, LLVMCCallConv);
  final List<String> parameterNames1 = functionNode1.getParameterNames();
  for (int i = 0; i < parameterNames1.size(); i++) {
    parameters.put(parameterNames1.get(i), LLVMGetParam(functionValueRef1, i));
    parameterTypes.put(parameterNames1.get(i), i32Type);
  }

  // 定义函数体
  // 创建一个基本块
  LLVMBasicBlockRef entryBlock1 = LLVMAppendBasicBlockInContext(context, functionValueRef1, "entry1");

  // 在基本块中添加指令
  LLVMPositionBuilderAtEnd(builder, entryBlock1);
  for (AStatement statementNode : functionNode1.getBlockNode().getStatementNodes()) {
    if (statementNode instanceof SDeclBlock) {
      SDeclBlock sDeclBlock = (SDeclBlock) statementNode;
      for (SDeclaration declarationNode : sDeclBlock.getDeclarationNodes()) {
        LLVMValueRef valueRef = null;
        final AExpression valueNode1 = declarationNode.getValueNode();
        if (valueNode1 instanceof ENumeric){
          final ENumeric valueNode = (ENumeric) valueNode1;
          if (declarationNode.getCanonicalTypeName().equals("def")){
            valueRef = LLVMBuildAlloca(builder, i32Type, declarationNode.getSymbol());
          }
          final LLVMValueRef valueRef1 = LLVMConstInt(i32Type, Long.parseLong(valueNode.getNumeric(), valueNode.getRadix()), 0);
          LLVMBuildStore(builder, valueRef1, valueRef);
          parameters.put(declarationNode.getSymbol(), valueRef);
          parameterTypes.put(declarationNode.getSymbol(), i32Type);
        }
        if (valueNode1 instanceof ENewArray) {

          ENewArray eNewArray = (ENewArray)valueNode1;
          LLVMTypeRef llvmTypeRef = null;
          if (declarationNode.getCanonicalTypeName().equals("int[]")){
            llvmTypeRef = LLVMArrayType(i32Type, 5);
            valueRef = LLVMBuildAlloca(builder, llvmTypeRef, declarationNode.getSymbol());
          }
          if (eNewArray.isInitializer()) {
            for (int i = 0; i < eNewArray.getValueNodes().size(); i++) {
              final AExpression valueNode = eNewArray.getValueNodes().get(i);
              if (valueNode instanceof ENumeric) {
                ENumeric eNumeric = (ENumeric) valueNode;
                PointerPointer<Pointer> indices = new PointerPointer<>(1);
                indices.put(0, LLVMConstInt(i32Type, i, 0));
                LLVMValueRef arrayElement = LLVMBuildGEP2(builder, llvmTypeRef, valueRef, indices, 1, "myArrayElement");
                LLVMBuildStore(builder, LLVMConstInt(i32Type, Long.parseLong(eNumeric.getNumeric()), 0), arrayElement);
              }
            }
          } else {
            LLVMBuildStore(builder, LLVMConstNull(llvmTypeRef), valueRef);
          }
          parameters.put(declarationNode.getSymbol(), valueRef);
          parameterTypes.put(declarationNode.getSymbol(), llvmTypeRef);
        }

      }
    }

    if (statementNode instanceof SFor) {
      SFor sFor = (SFor)statementNode;
      LLVMBasicBlockRef preheaderBB = LLVMAppendBasicBlock(functionValueRef1, "preheader");
      LLVMBasicBlockRef continueBB = LLVMAppendBasicBlock(functionValueRef1, "continue");
      LLVMBasicBlockRef loopBB = LLVMAppendBasicBlock(functionValueRef1, "loop");
      LLVMBasicBlockRef entryBB = LLVMAppendBasicBlock(functionValueRef1, "entry");
      LLVMBasicBlockRef exitBB = LLVMAppendBasicBlock(functionValueRef1, "exit");

      // 在当前函数的入口处创建分支指令，将控制流转移到 preheaderBB
      LLVMBuildBr(builder, preheaderBB);

      // 在 preheaderBB 基本块中插入代码，初始化循环计数器
      LLVMPositionBuilderAtEnd(builder, preheaderBB);
      LLVMValueRef i = LLVMBuildAlloca(builder, i32Type, "i");
      LLVMBuildStore(builder, LLVMConstInt(i32Type, 0, 0), i);

      // 在 loopBB 基本块中插入代码，实现循环逻辑
      LLVMBuildBr(builder, loopBB);
      LLVMPositionBuilderAtEnd(builder, loopBB);
      LLVMValueRef count = LLVMBuildLoad2(builder, i32Type, i, "count");
      LLVMValueRef compare = LLVMBuildICmp(builder, LLVMIntSLT, count, LLVMConstInt(i32Type, 5, 0), "compare");
      LLVMBuildCondBr(builder, compare, entryBB, exitBB);

      LLVMPositionBuilderAtEnd(builder, entryBB);

      // for 代码块内部逻辑
      PointerPointer<Pointer> indices11 = new PointerPointer<>(1);
      indices11.put(0, LLVMBuildLoad2(builder, i32Type, i, "i"));
      LLVMValueRef arrayElement11 = LLVMBuildGEP2(builder, parameterTypes.get("b1"), parameters.get("b1"), indices11, 1, "myArrayElement");
      final LLVMValueRef b1Value = LLVMBuildLoad2(builder, i32Type, arrayElement11, "b1[]");

      PointerPointer<Pointer> args = new PointerPointer<>(1);
      args.put(0, b1Value);
      LLVMValueRef result = LLVMBuildCall2(builder, LLVMFunctionType(i32Type, argumentTypes, paramsSize, 0), functionValueRef, args, 1, "result");

      final LLVMValueRef valueRef = LLVMBuildAdd(builder, LLVMBuildLoad2(builder, parameterTypes.get("total"), parameters.get("total"),"loadTotal"), result, "add");

      LLVMBuildStore(builder, valueRef, parameters.get("total"));

      LLVMBuildBr(builder, continueBB);

      LLVMPositionBuilderAtEnd(builder, continueBB);
      LLVMValueRef inc = LLVMBuildAdd(builder, count, LLVMConstInt(i32Type, 1, 0), "inc");
      LLVMBuildStore(builder, inc, i);

      LLVMBuildBr(builder, loopBB);

      // 在 exitBB 基本块中插入代码，完成循环后的逻辑
      LLVMPositionBuilderAtEnd(builder, exitBB);

    }

    if (statementNode instanceof SReturn) {
      SReturn sReturn = (SReturn)statementNode;
      final ESymbol valueNode = (ESymbol)sReturn.getValueNode();
      LLVMBuildRet(builder, LLVMBuildLoad2(builder, parameterTypes.get(valueNode.getSymbol()),parameters.get(valueNode.getSymbol()), "ret"));
    }
  }

  BytePointer error = new BytePointer();
  LLVMDumpModule(module);
  if (LLVMVerifyModule(module, LLVMPrintMessageAction, error) != 0) {
    System.out.println("Failed to validate module: " + error.getString());
    return;
  }

  // Stage 3: Dump the module to file
  //if (LLVMWriteBitcodeToFile(module, "./demo.bc") != 0) {
  //    System.err.println("Failed to write bitcode to file");
  //    return;
  //}
  BytePointer out = new BytePointer((Pointer)null);
  if (LLVMPrintModuleToFile(module, "./demo.ll" ,out) != 0) {
    System.err.println("Failed to write ll to file");
    return;
  }

  // Stage 5: Dispose of allocated resources
  LLVMDisposeBuilder(builder);
  LLVMDisposeModule(module);
  LLVMContextDispose(context);
}

public void getLLVM_DEBUG(SClass root, ClassNode classNode, String source) {
  // Stage 1: Initialize LLVM components
  LLVMInitializeCore(LLVMGetGlobalPassRegistry());
  LLVMInitializeNativeAsmPrinter();
  LLVMInitializeNativeAsmParser();
  LLVMInitializeNativeDisassembler();
  LLVMInitializeNativeTarget();
  // Stage 2: Build the sum function
  LLVMOrcThreadSafeContextRef threadContext = LLVMOrcCreateNewThreadSafeContext();
  LLVMContextRef context = LLVMOrcThreadSafeContextGetContext(threadContext);
  LLVMModuleRef module = LLVMModuleCreateWithNameInContext(root.getLocation().getSourceName(), context);
  LLVMBuilderRef builder = LLVMCreateBuilderInContext(context);
  LLVMTypeRef i32Type = LLVMInt32TypeInContext(context);

  //for (int index = 0; index < root.getFunctionNodes().size(); index++) {
  final SFunction functionNode = root.getFunctionNodes().get(0);
  LLVMValueRef functionValueRef;
  // 定义函数及其对应的参数配置
  final int paramsSize = functionNode.getCanonicalTypeNameParameters().size();
  PointerPointer<Pointer> argumentTypes = new PointerPointer<>(paramsSize);
  for (int i = 0; i < paramsSize; i++) {
    argumentTypes.put(i, i32Type);
  }
  functionValueRef = LLVMAddFunction(module, functionNode.getFunctionName(), LLVMFunctionType(i32Type, argumentTypes, paramsSize, 0)); // LLVMGetTypeByName2(context, functionNode.getReturnCanonicalTypeName())
  LLVMSetFunctionCallConv(functionValueRef, LLVMCCallConv);
  LLVMSetLinkage(functionValueRef, LLVMExternalLinkage);
  final List<String> parameterNames = functionNode.getParameterNames();
  Map<String, LLVMValueRef> parameters = new HashMap<>(parameterNames.size());
  for (int i = 0; i < parameterNames.size(); i++) {
    parameters.put(parameterNames.get(i), LLVMGetParam(functionValueRef, i));
  }
  // 定义函数体
  // 创建一个基本块
  LLVMBasicBlockRef entryBlock = LLVMAppendBasicBlockInContext(context, functionValueRef, "entry");

  // 在基本块中添加指令
  LLVMPositionBuilderAtEnd(builder, entryBlock);

  /*****************************调试代码部分开始************************************/

  // 初始化 -1
  final LLVMValueRef valueRef1 = LLVMBuildAlloca(builder, i32Type, "ret");

  final LLVMValueRef valueRef = LLVMBuildSub(builder, LLVMConstInt(i32Type, 0, 0), LLVMConstInt(i32Type, 1, 0), "-1");

  LLVMBuildStore(builder, valueRef, valueRef1);

  // ---
  // 取余操作
  final LLVMValueRef valueRef2 = LLVMBuildAlloca(builder, i32Type, "test");

  final LLVMValueRef valueRef3 = LLVMBuildSRem(builder, parameters.get("x"), LLVMConstInt(i32Type, 10, 0), "x%10");

  LLVMBuildStore(builder, valueRef3, valueRef2);

  // 创建局部数组
  final LLVMTypeRef llvmTypeRef = LLVMArrayType(i32Type, 4);
  final LLVMValueRef alloc = LLVMBuildAlloca(builder, llvmTypeRef, "myArray");
  LLVMBuildStore(builder, LLVMConstNull(llvmTypeRef), alloc);
  // 加载和存储数组元素的值
  PointerPointer<Pointer> indices = new PointerPointer<>(1);
  indices.put(0, LLVMConstInt(i32Type, 2, 0));
  LLVMValueRef arrayElement = LLVMBuildGEP2(builder, llvmTypeRef, alloc, indices, 1, "myArrayElement");
  LLVMBuildStore(builder, LLVMConstInt(i32Type, 2, 0), arrayElement);

  // if else
  /*
        示例代码中，我们首先定义了一个名为“main”的函数，并在该函数中定义了一个整型变量“x”，将其初始化为5。
        然后我们定义了三个基本块，分别代表if块、else块和结束块，并使用“LLVMBuildICmp”函数比较x和5的大小。
        然后我们使用“LLVMBuildCondBr”函数基于比较结果跳转到if块或else块。在if块中，我们调用“LLVMBuildCall”函数，并传递x的地址作为参数。
        在else块中，我们将x的值修改为10。最后，我们返回void类型，并将生成的模块打印到标准输出。
         */
  LLVMValueRef x = LLVMBuildAlloca(builder, i32Type, "x");
  LLVMBuildStore(builder, LLVMConstInt(i32Type, 5, 0), x);

  LLVMBasicBlockRef ifBlock = LLVMAppendBasicBlock(functionValueRef, "ifBlock");
  LLVMBasicBlockRef elseBlock = LLVMAppendBasicBlock(functionValueRef, "elseBlock");
  LLVMBasicBlockRef endBlock = LLVMAppendBasicBlock(functionValueRef, "endBlock");

  LLVMValueRef cmp = LLVMBuildICmp(builder, LLVMIntEQ, LLVMBuildLoad2(builder, i32Type, x, "load"), LLVMConstInt(i32Type, 5, 0), "cmp");

  LLVMValueRef cond1 = LLVMBuildICmp(builder, LLVMIntSGT, LLVMConstInt(i32Type, 5, 0), LLVMConstInt(i32Type, 5, 0), "cond1");
  LLVMValueRef cond2 = LLVMBuildICmp(builder, LLVMIntSLE, LLVMConstInt(i32Type, 5, 0), LLVMConstInt(i32Type, 5, 0), "cond2");
  LLVMValueRef andCond = LLVMBuildAnd(builder, cond1, cond2, "andCond");
  LLVMValueRef andCond1 = LLVMBuildAnd(builder, andCond, cmp, "andCond1");

  LLVMBuildCondBr(builder, andCond1, ifBlock, elseBlock);

  LLVMPositionBuilderAtEnd(builder, ifBlock);
  //LLVMBuildCall(builder, LLVMGetNamedFunction(LLVMGetModuleContext(LLVMGetBasicBlockParent(LLVMGetInsertBlock(builder)))), new PointerPointer<LLVMValueRef>(x), 1, "ifCall");
  PointerPointer<Pointer> indices1 = new PointerPointer<>(1);
  indices1.put(0, LLVMConstInt(i32Type, 2, 0));
  LLVMValueRef arrayElement1 = LLVMBuildGEP2(builder, llvmTypeRef, alloc, indices1, 1, "myArrayElement");
  LLVMBuildStore(builder, arrayElement1, x);
  LLVMBuildBr(builder, endBlock);

  LLVMPositionBuilderAtEnd(builder, elseBlock);
  LLVMBuildStore(builder, LLVMConstInt(LLVMInt32Type(), 10, 0), x);
  LLVMBuildBr(builder, endBlock);

  LLVMPositionBuilderAtEnd(builder, endBlock);

  // for
  /*
            代码演示了一个简单的 for 循环，循环次数为 10 次。
            在该示例中，通过三个基本块实现了循环：preheaderBB、loopBB 和 exitBB。preheaderBB 是入口基本块，用于初始化循环计数器 i。
            loopBB 是循环体，执行循环逻辑，并将计数器 i 加 1，直到 i 小于 10。exitBB 是循环结束后的基本块。
         */
  LLVMBasicBlockRef preheaderBB = LLVMAppendBasicBlock(functionValueRef, "preheader");
  LLVMBasicBlockRef loopBB = LLVMAppendBasicBlock(functionValueRef, "loop");
  LLVMBasicBlockRef exitBB = LLVMAppendBasicBlock(functionValueRef, "exit");

  // 在当前函数的入口处创建分支指令，将控制流转移到 preheaderBB
  LLVMBuildBr(builder, preheaderBB);

  // 在 preheaderBB 基本块中插入代码，初始化循环计数器
  LLVMPositionBuilderAtEnd(builder, preheaderBB);
  LLVMValueRef i = LLVMBuildAlloca(builder, i32Type, "i");
  LLVMBuildStore(builder, LLVMConstInt(i32Type, 0, 0), i);

  // 在 loopBB 基本块中插入代码，实现循环逻辑
  LLVMBuildBr(builder, loopBB);
  LLVMPositionBuilderAtEnd(builder, loopBB);
  LLVMValueRef count = LLVMBuildLoad2(builder, i32Type, i, "count");
  LLVMValueRef compare = LLVMBuildICmp(builder, LLVMIntSLT, count, LLVMConstInt(i32Type, 10, 0), "compare");
  LLVMBasicBlockRef continueBB = LLVMAppendBasicBlock(functionValueRef, "continue");
  LLVMBuildCondBr(builder, compare, continueBB, exitBB);
  LLVMPositionBuilderAtEnd(builder, continueBB);
  LLVMValueRef inc = LLVMBuildAdd(builder, count, LLVMConstInt(i32Type, 1, 0), "inc");
  LLVMBuildStore(builder, inc, i);
  LLVMBuildBr(builder, loopBB);

  // 在 exitBB 基本块中插入代码，完成循环后的逻辑
  LLVMPositionBuilderAtEnd(builder, exitBB);

  LLVMBuildRet(builder, LLVMConstInt(i32Type, 0, 0));


  // 函数调用
  LLVMValueRef functionValueRef1 = LLVMAddFunction(module, "exec", LLVMFunctionType(i32Type, new PointerPointer<>(0), 0, 0)); // LLVMGetTypeByName2(context, functionNode.getReturnCanonicalTypeName())
  LLVMSetFunctionCallConv(functionValueRef1, LLVMCCallConv);
  // 创建一个基本块
  LLVMBasicBlockRef entryBlock1 = LLVMAppendBasicBlockInContext(context, functionValueRef1, "entry1");

  // 在基本块中添加指令
  LLVMPositionBuilderAtEnd(builder, entryBlock1);
  // Create the function call instruction
  PointerPointer<Pointer> args = new PointerPointer<>(1);
  args.put(0, LLVMConstInt(i32Type, 1, 0));
  LLVMValueRef result = LLVMBuildCall2(builder, LLVMFunctionType(i32Type, argumentTypes, paramsSize, 0), functionValueRef, args, 1, "result");
  //LLVMValueRef result = LLVMBuildCall2(builder, LLVMTypeOf(functionValueRef), functionValueRef, args, 1, "result");

  // Create the return instruction
  LLVMBuildRet(builder, result);


  /******************************调试代码部分结束***********************************/

  //}


  BytePointer error = new BytePointer();
  LLVMDumpModule(module);
  if (LLVMVerifyModule(module, LLVMPrintMessageAction, error) != 0) {
    System.out.println("Failed to validate module: " + error.getString());
    return;
  }

  // Stage 3: Dump the module to file
  //if (LLVMWriteBitcodeToFile(module, "./demo.bc") != 0) {
  //    System.err.println("Failed to write bitcode to file");
  //    return;
  //}
  BytePointer out = new BytePointer((Pointer)null);
  if (LLVMPrintModuleToFile(module, "./demo.ll" ,out) != 0) {
    System.err.println("Failed to write ll to file");
    return;
  }

  // Stage 5: Dispose of allocated resources
  LLVMDisposeBuilder(builder);
  LLVMDisposeModule(module);
  LLVMContextDispose(context);
}
```

