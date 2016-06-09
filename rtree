package cz.rtree;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;

public final class RTree {

    final static private int MAX_LEVEL = 28;
    final static private int MAX_RECT_OBJ_COUNT = 50;
    final static private int PREALLOC_SIZE = 51;
    // Global Rect:
    final static public int GLOBAL_MIN_X = -180000000;
    final static public int GLOBAL_MIN_Y = -90000000;
    final static public int GLOBAL_MAX_X = 180000000;
    final static public int GLOBAL_MAX_Y = 90000000;
    // Rect constants:
    final static private int IDX_RECT_SUB1_OFFS = 0;
    final static private int IDX_RECT_SUB2_OFFS = 1;
    final static private int IDX_RECT_OBJARR_OFFS = 2;
    final static private int IDX_RECT_OBJARR_COUNT_OFFS = 3;
    final static public int IDX_RECT_SIZE = 4;
    // Object constants:
    final static private int IDX_OBJ_ID_OFFS = 0;
    final static private int IDX_OBJ_MINX_OFFS = 1;
    final static private int IDX_OBJ_MINY_OFFS = 2;
    final static private int IDX_OBJ_MAXX_OFFS = 3;
    final static private int IDX_OBJ_MAXY_OFFS = 4;
    final static private int IDX_OBJ_RECT_OFFS = 5;
    final static public int IDX_OBJ_SIZE = 6;
    // Main Arrays:
    private int[] rects;
    private int[] objs;
    private int[] objsCompact;
    private int nextRect;
    private int nextObj;
    private boolean compactMode;
    private int incrementalAdd;

    // Normal constructor for empty R-Tree
    public RTree(int maxRectCount, int maxObjCount) {
        compactMode = false;
        if (maxRectCount < 10000) {
            maxRectCount = 10000;
        }
        if (maxObjCount < 100000) {
            maxObjCount = 100000;
        }
        rects = new int[maxRectCount * IDX_RECT_SIZE];
        objs = new int[maxObjCount * IDX_OBJ_SIZE];
        nextRect = 0;
        nextObj = 0;
        addGlobalZeroRect();
    }

    // Constructor for pre-loaded R-Tree
    public RTree(int[] rectsInp, int[] objsInp, boolean compactReadOnly) {
        compactMode = compactReadOnly;
        rects = rectsInp;
        if (compactReadOnly) {
            objsCompact = objsInp;
            nextObj = objsCompact.length;
        } else {
            objs = objsInp;
            nextObj = objs.length / IDX_OBJ_SIZE;
        }
        nextRect = rects.length / IDX_RECT_SIZE;        
    }

    public int getRectCount() {
        return nextRect;
    }

    public int getObjCount() {
        return nextObj;
    }

    public int[] getRectArr() {
        return rects;
    }

    public int[] getObjArr() {
        return (compactMode ? objsCompact : objs);
    }

    public void turnToCompactMode() {
        if (compactMode) {
            return;
        }
        shrink();
        objsCompact = new int[nextObj];
        for (int i = 0; i < objsCompact.length; i++) {
            objsCompact[i] = objs[i * IDX_OBJ_SIZE];
        }
        objs = null;
        compactMode = true;
    }

    public void addObj(int id, int minX, int minY, int maxX, int maxY) {
        if (compactMode) {
            return; // Error: insertion not possible in read-only mode
        }
        if ((++incrementalAdd % 100) == 0) {
            checkObjsCapacity();
            checkRectsCapacity();
        }
        if (minX < GLOBAL_MIN_X) {
            minX = GLOBAL_MIN_X;
        }
        if (maxX > GLOBAL_MAX_X) {
            maxX = GLOBAL_MAX_X;
        }
        if (minY < GLOBAL_MIN_Y) {
            minY = GLOBAL_MIN_Y;
        }
        if (maxY > GLOBAL_MAX_Y) {
            maxY = GLOBAL_MAX_Y;
        }
        if (minX > maxX) {
            maxX = minX;
        }
        if (minY > maxY) {
            maxY = minY;
        }
        addObjToRect(id, minX, minY, maxX, maxY, 0, 0, GLOBAL_MIN_X, GLOBAL_MIN_Y, GLOBAL_MAX_X, GLOBAL_MAX_Y);
    }

    private int addGlobalZeroRect() {
        int offs = nextRect * IDX_RECT_SIZE;
        rects[offs + IDX_RECT_SUB1_OFFS] = -1;
        rects[offs + IDX_RECT_SUB2_OFFS] = -1;
        rects[offs + IDX_RECT_OBJARR_COUNT_OFFS] = 0;
        return nextRect++;
    }

    private void addObjToRect(int id, int minX, int minY, int maxX, int maxY,
            int rectNr, int level, int rectMinX, int rectMinY, int rectMaxX, int rectMaxY) {
        if ((minX > rectMaxX) || (maxX < rectMinX) || (minY > rectMaxY) || (maxY < rectMinY)) {
            return;
        }
        int sub1MinX, sub1MinY, sub1MaxX, sub1MaxY;
        int sub2MinX, sub2MinY, sub2MaxX, sub2MaxY;
        int dx = rectMaxX - rectMinX;
        int dy = rectMaxY - rectMinY;
        if (dx >= dy) {
            int midX = (rectMinX + rectMaxX) / 2;
            sub1MinX = rectMinX;
            sub1MinY = rectMinY;
            sub1MaxX = midX;
            sub1MaxY = rectMaxY;
            sub2MinX = midX + 1;
            sub2MinY = rectMinY;
            sub2MaxX = rectMaxX;
            sub2MaxY = rectMaxY;
        } else {
            int midY = (rectMinY + rectMaxY) / 2;
            sub1MinX = rectMinX;
            sub1MinY = rectMinY;
            sub1MaxX = rectMaxX;
            sub1MaxY = midY + 1;
            sub2MinX = rectMinX;
            sub2MinY = midY;
            sub2MaxX = rectMaxX;
            sub2MaxY = rectMaxY;
        }
        int offs = rectNr * IDX_RECT_SIZE;
        int sub1 = rects[offs + IDX_RECT_SUB1_OFFS];
        int sub2 = rects[offs + IDX_RECT_SUB2_OFFS];
        if ((sub1 != -1) || (sub2 != -1)) {
            if (sub1 != -1) {
                addObjToRect(id, minX, minY, maxX, maxY, sub1, level + 1, sub1MinX, sub1MinY, sub1MaxX, sub1MaxY);
            }
            if (sub2 != -1) {
                addObjToRect(id, minX, minY, maxX, maxY, sub2, level + 1, sub2MinX, sub2MinY, sub2MaxX, sub2MaxY);
            }
            return;
        }
        int objIdx = rects[offs + IDX_RECT_OBJARR_OFFS];
        int objCount = rects[offs + IDX_RECT_OBJARR_COUNT_OFFS];
        int enlargePos = objIdx + objCount;
        int newStart;
        if ((enlargePos < nextObj) && isFreeObj(enlargePos)) {
            newStart = objIdx;
            createNewObjectOnIndex(id, minX, minY, maxX, maxY, rectNr, enlargePos);
        } else {
            newStart = createNewObject(id, minX, minY, maxX, maxY, rectNr);
            if (objCount > 0) {
                bringObjectsToFront(objIdx, objCount, newStart + 1);
                clearObjects(objIdx, objCount);
            }
            rects[offs + IDX_RECT_OBJARR_OFFS] = newStart;
        }
        objCount++;
        rects[offs + IDX_RECT_OBJARR_COUNT_OFFS] = objCount;

        if ((objCount > MAX_RECT_OBJ_COUNT) && (level < MAX_LEVEL)) {
            sub1 = createSubRect(newStart, objCount, sub1MinX, sub1MinY, sub1MaxX, sub1MaxY, level + 1);
            sub2 = createSubRect(newStart, objCount, sub2MinX, sub2MinY, sub2MaxX, sub2MaxY, level + 1);
            rects[offs + IDX_RECT_SUB1_OFFS] = sub1;
            rects[offs + IDX_RECT_SUB2_OFFS] = sub2;
            if ((sub1 != -1) || (sub2 != -1)) {
                clearObjects(newStart, objCount);
            }
        }
    }

    private boolean isFreeObj(int obj) {
        int offs = obj * IDX_OBJ_SIZE;
        return (objs[offs + IDX_OBJ_RECT_OFFS] == -1);
    }

    private void clearObjects(int firstObject, int count) {
        for (int i = firstObject; i < (firstObject + count); i++) {
            int objOffs = i * IDX_OBJ_SIZE;
            objs[objOffs + IDX_OBJ_RECT_OFFS] = -1;
        }
    }

    private int createNewObject(int id, int minX, int minY, int maxX, int maxY, int rectNr) {
        int offs = nextObj * IDX_OBJ_SIZE;
        objs[offs + IDX_OBJ_RECT_OFFS] = rectNr;
        objs[offs + IDX_OBJ_ID_OFFS] = id;
        objs[offs + IDX_OBJ_MINX_OFFS] = minX;
        objs[offs + IDX_OBJ_MINY_OFFS] = minY;
        objs[offs + IDX_OBJ_MAXX_OFFS] = maxX;
        objs[offs + IDX_OBJ_MAXY_OFFS] = maxY;
        int localNextObj = nextObj++;
        incNextObj();
        return localNextObj;
    }

    private void createNewObjectOnIndex(int id, int minX, int minY, int maxX, int maxY, int rectNr, int pos) {
        int offs = pos * IDX_OBJ_SIZE;
        objs[offs + IDX_OBJ_RECT_OFFS] = rectNr;
        objs[offs + IDX_OBJ_ID_OFFS] = id;
        objs[offs + IDX_OBJ_MINX_OFFS] = minX;
        objs[offs + IDX_OBJ_MINY_OFFS] = minY;
        objs[offs + IDX_OBJ_MAXX_OFFS] = maxX;
        objs[offs + IDX_OBJ_MAXY_OFFS] = maxY;
    }

    private void incNextObj() {
        for (int i = 0; i < PREALLOC_SIZE; i++) {
            int offs = (nextObj + i) * IDX_OBJ_SIZE;
            objs[offs + IDX_OBJ_RECT_OFFS] = -1;
        }
        nextObj += PREALLOC_SIZE;
    }

    private void bringObjectsToFront(int objIdx, int objCount, int pos) {
        int offs1 = objIdx * IDX_OBJ_SIZE;
        int offs2 = pos * IDX_OBJ_SIZE;
        int sz = objCount * IDX_OBJ_SIZE;
        for (int i = 0; i < sz; i++) {
            objs[offs2 + i] = objs[offs1 + i];
        }
        int end = pos + objCount;
        if (end > nextObj) {
            nextObj = end;
            incNextObj();
        }
    }

    private int bringObjectsToFrontFiltered(int firstObj, int objCount, int newRecNr, int rectMinX, int rectMinY, int rectMaxX, int rectMaxY) {
        int count = 0;
        for (int i = 0; i < objCount; i++) {
            int offs = (firstObj + i) * IDX_OBJ_SIZE;
            int minX = objs[offs + IDX_OBJ_MINX_OFFS];
            int minY = objs[offs + IDX_OBJ_MINY_OFFS];
            int maxX = objs[offs + IDX_OBJ_MAXX_OFFS];
            int maxY = objs[offs + IDX_OBJ_MAXY_OFFS];
            if ((minX <= rectMaxX) && (maxX >= rectMinX) && (minY <= rectMaxY) && (maxY >= rectMinY)) {
                objs[offs + IDX_OBJ_RECT_OFFS] = newRecNr;
                copyObject(firstObj + i, nextObj++);
                count++;
            }
        }
        incNextObj();
        return count;
    }

    private boolean anyObjNotFullyCovering(int firstObj, int objCount, int rectMinX, int rectMinY, int rectMaxX, int rectMaxY) {
        for (int i = 0; i < objCount; i++) {
            int offs = (firstObj + i) * IDX_OBJ_SIZE;
            int minX = objs[offs + IDX_OBJ_MINX_OFFS];
            int minY = objs[offs + IDX_OBJ_MINY_OFFS];
            int maxX = objs[offs + IDX_OBJ_MAXX_OFFS];
            int maxY = objs[offs + IDX_OBJ_MAXY_OFFS];
            if ((minX > rectMinX) || (maxX < rectMaxX) || (minY > rectMinY) || (maxY < rectMaxY)) {
                return true;
            }
        }
        return false;
    }

    private int countObjInside(int firstObj, int objCount, int rectMinX, int rectMinY, int rectMaxX, int rectMaxY) {
        int count = 0;
        for (int i = 0; i < objCount; i++) {
            int offs = (firstObj + i) * IDX_OBJ_SIZE;
            int minX = objs[offs + IDX_OBJ_MINX_OFFS];
            int minY = objs[offs + IDX_OBJ_MINY_OFFS];
            int maxX = objs[offs + IDX_OBJ_MAXX_OFFS];
            int maxY = objs[offs + IDX_OBJ_MAXY_OFFS];
            if (!((minX > rectMaxX) || (maxX < rectMinX) || (minY > rectMaxY) || (maxY < rectMinY))) {
                count++;
            }
        }
        return count;
    }

    private int createSubRect(int firstObj, int objCount, int rectMinX, int rectMinY, int rectMaxX, int rectMaxY, int level) {
        int countInside = countObjInside(firstObj, objCount, rectMinX, rectMinY, rectMaxX, rectMaxY);
        /*
        if (countInside == 0) {
            return -1;
        }
        */

        int localNextRec = nextRect++;
        int offs = localNextRec * IDX_RECT_SIZE;

        boolean continueRecursion = ((countInside > 0) && anyObjNotFullyCovering(firstObj, objCount, rectMinX, rectMinY, rectMaxX, rectMaxY));

        int sub1 = -1;
        int sub2 = -1;
        int sub1MinX, sub1MinY, sub1MaxX, sub1MaxY;
        int sub2MinX, sub2MinY, sub2MaxX, sub2MaxY;
        int dx = rectMaxX - rectMinX;
        int dy = rectMaxY - rectMinY;
        if ((level < MAX_LEVEL) && (countInside > MAX_RECT_OBJ_COUNT) && continueRecursion) {
            if (dx >= dy) {
                int midX = (rectMinX + rectMaxX) / 2;
                sub1MinX = rectMinX;
                sub1MinY = rectMinY;
                sub1MaxX = midX;
                sub1MaxY = rectMaxY;
                sub2MinX = midX + 1;
                sub2MinY = rectMinY;
                sub2MaxX = rectMaxX;
                sub2MaxY = rectMaxY;
            } else {
                int midY = (rectMinY + rectMaxY) / 2;
                sub1MinX = rectMinX;
                sub1MinY = rectMinY;
                sub1MaxX = rectMaxX;
                sub1MaxY = midY + 1;
                sub2MinX = rectMinX;
                sub2MinY = midY;
                sub2MaxX = rectMaxX;
                sub2MaxY = rectMaxY;
            }
            sub1 = createSubRect(firstObj, objCount, sub1MinX, sub1MinY, sub1MaxX, sub1MaxY, level + 1);
            sub2 = createSubRect(firstObj, objCount, sub2MinX, sub2MinY, sub2MaxX, sub2MaxY, level + 1);
        }
        rects[offs + IDX_RECT_SUB1_OFFS] = sub1;
        rects[offs + IDX_RECT_SUB2_OFFS] = sub2;
        if ((sub1 != -1) || (sub2 != -1)) {
            rects[offs + IDX_RECT_OBJARR_COUNT_OFFS] = 0;
            clearObjects(firstObj, objCount);
        } else {
            rects[offs + IDX_RECT_OBJARR_OFFS] = nextObj;
            int newCount = bringObjectsToFrontFiltered(firstObj, objCount, localNextRec, rectMinX, rectMinY, rectMaxX, rectMaxY);
            rects[offs + IDX_RECT_OBJARR_COUNT_OFFS] = newCount;
        }
        return localNextRec;
    }

    private void copyObject(int from, int to) {
        int offs1 = from * IDX_OBJ_SIZE;
        int offs2 = to * IDX_OBJ_SIZE;
        for (int i = 0; i < IDX_OBJ_SIZE; i++) {
            objs[offs2 + i] = objs[offs1 + i];
        }
    }

    private void moveObjects(int from, int count, int to) {
        for (int i = 0; i < count; i++) {
            int offs1 = (from + i) * IDX_OBJ_SIZE;
            int offs2 = (to + i) * IDX_OBJ_SIZE;
            for (int j = 0; j < IDX_OBJ_SIZE; j++) {
                objs[offs2 + j] = objs[offs1 + j];
            }
            objs[offs1 + IDX_OBJ_RECT_OFFS] = -1;
        }
    }

    public void shrink() {
        int nextFreeObj = findFreeObj(0);
        if (nextFreeObj < 0) {
            return;
        }
        int newNextObj = 0;
        int findFilled = nextFreeObj + 1;
        while (findFilled < nextObj) {
            int rect = objs[(findFilled * IDX_OBJ_SIZE) + IDX_OBJ_RECT_OFFS];
            if (rect == -1) {
                findFilled++;
            } else {
                int count = rects[(rect * IDX_RECT_SIZE) + IDX_RECT_OBJARR_COUNT_OFFS];
                moveObjects(findFilled, count, nextFreeObj);
                rects[(rect * IDX_RECT_SIZE) + IDX_RECT_OBJARR_OFFS] = nextFreeObj;
                newNextObj = (nextFreeObj + count);
                nextFreeObj = findFreeObj(nextFreeObj + count);
                findFilled += count;
            }
        }
        if (newNextObj > 0) {
            nextObj = newNextObj;
        }
    }

    private int findFreeObj(int firstObj) {
        for (int i = firstObj; i < nextObj; i++) {
            int offs = i * IDX_OBJ_SIZE;
            if (objs[offs + IDX_OBJ_RECT_OFFS] == -1) {
                return i;
            }
        }
        return -1;
    }

    public ArrayList<Integer> getIntersectingObjs(int minX, int minY, int maxX, int maxY, boolean removeDuplicates) {
        ArrayList<Integer> res = new ArrayList<>(200);
        int rectMinX = GLOBAL_MIN_X;
        int rectMinY = GLOBAL_MIN_Y;
        int rectMaxX = GLOBAL_MAX_X;
        int rectMaxY = GLOBAL_MAX_Y;
        int rectNr = 0;
        addIntersectingObjs(res, rectNr, minX, minY, maxX, maxY, rectMinX, rectMinY, rectMaxX, rectMaxY);
        if (removeDuplicates) {
            Collections.sort(res);
            int prevObjId = Integer.MAX_VALUE;
            for (int i = res.size() - 1; i >= 0; i--) {
                int val = res.get(i);
                if (val == prevObjId) {
                    res.remove(i);
                } else {
                    prevObjId = val;
                }
            }
        }
        return res;
    }

    private void addIntersectingObjs(ArrayList<Integer> objIdArr, int rectNr, int minX, int minY, int maxX, int maxY,
            int rectMinX, int rectMinY, int rectMaxX, int rectMaxY) {
        if (((minX > rectMaxX) || (maxX < rectMinX) || (minY > rectMaxY) || (maxY < rectMinY))) {
            return;
        }
        int offs = rectNr * IDX_RECT_SIZE;
        int sub1 = rects[offs + IDX_RECT_SUB1_OFFS];
        int sub2 = rects[offs + IDX_RECT_SUB2_OFFS];
        if ((sub1 != -1) || (sub2 != -1)) {
            int sub1MinX, sub1MinY, sub1MaxX, sub1MaxY;
            int sub2MinX, sub2MinY, sub2MaxX, sub2MaxY;
            int dx = rectMaxX - rectMinX;
            int dy = rectMaxY - rectMinY;
            if (dx >= dy) {
                int midX = (rectMinX + rectMaxX) / 2;
                sub1MinX = rectMinX;
                sub1MinY = rectMinY;
                sub1MaxX = midX;
                sub1MaxY = rectMaxY;
                sub2MinX = midX + 1;
                sub2MinY = rectMinY;
                sub2MaxX = rectMaxX;
                sub2MaxY = rectMaxY;
            } else {
                int midY = (rectMinY + rectMaxY) / 2;
                sub1MinX = rectMinX;
                sub1MinY = rectMinY;
                sub1MaxX = rectMaxX;
                sub1MaxY = midY + 1;
                sub2MinX = rectMinX;
                sub2MinY = midY;
                sub2MaxX = rectMaxX;
                sub2MaxY = rectMaxY;
            }
            if (sub1 != -1) {
                addIntersectingObjs(objIdArr, sub1, minX, minY, maxX, maxY, sub1MinX, sub1MinY, sub1MaxX, sub1MaxY);
            }
            if (sub2 != -1) {
                addIntersectingObjs(objIdArr, sub2, minX, minY, maxX, maxY, sub2MinX, sub2MinY, sub2MaxX, sub2MaxY);
            }
        } else {
            int firstObj = rects[offs + IDX_RECT_OBJARR_OFFS];
            int count = rects[offs + IDX_RECT_OBJARR_COUNT_OFFS];
            if (!compactMode) {
                for (int i = 0; i < count; i++) {
                    int objOffs = (firstObj + i) * IDX_OBJ_SIZE;
                    int objMinX = objs[objOffs + IDX_OBJ_MINX_OFFS];
                    int objMinY = objs[objOffs + IDX_OBJ_MINY_OFFS];
                    int objMaxX = objs[objOffs + IDX_OBJ_MAXX_OFFS];
                    int objMaxY = objs[objOffs + IDX_OBJ_MAXY_OFFS];
                    if (!((minX > objMaxX) || (maxX < objMinX) || (minY > objMaxY) || (maxY < objMinY))) {
                        int id = objs[objOffs + IDX_OBJ_ID_OFFS];
                        objIdArr.add(id);
                    }
                }
            } else { // compactMode
                for (int i = 0; i < count; i++) {
                    objIdArr.add(objsCompact[firstObj + i]);
                }
            }
        }
    }

    public boolean ASSERTCheckArraysConsistancy() {
        if (compactMode) {
            return true;
        }
        boolean[] rectChecked = new boolean[nextRect];
        for (int i = 0; i < nextObj; i++) {
            int objOffs = i * IDX_OBJ_SIZE;
            int rectNr = objs[objOffs + IDX_OBJ_RECT_OFFS];
            if (rectNr == -1) {
                continue;
            }
            rectChecked[rectNr] = true;
            int id = objs[objOffs + IDX_OBJ_ID_OFFS];
            int rectOffs = rectNr * IDX_RECT_SIZE;
            int firstObj = rects[rectOffs + IDX_RECT_OBJARR_OFFS];
            int count = rects[rectOffs + IDX_RECT_OBJARR_COUNT_OFFS];
            if ((i < firstObj) || (i >= (firstObj + count))) {
                System.out.println("Error, obj ID: " + id + " idx: " + i + " Rect: " + rectNr);
                return false;
            }
        }
        for (int i = 0; i < rectChecked.length; i++) {
            int rectOffs = i * IDX_RECT_SIZE;
            int count = rects[rectOffs + IDX_RECT_OBJARR_COUNT_OFFS];
            if (rectChecked[i]) {
                int firstObj = rects[rectOffs + IDX_RECT_OBJARR_OFFS];
                for (int j = firstObj; j < (firstObj + count); j++) {
                    int objRect = objs[(j * IDX_OBJ_SIZE) + IDX_OBJ_RECT_OFFS];
                    if (objRect != i) {
                        System.out.println("Error, obj idx: " + j + ", Rect: " + i);
                        return false;
                    }
                }
            } else { // rect not checked
                int sub1 = rects[rectOffs + IDX_RECT_SUB1_OFFS];
                int sub2 = rects[rectOffs + IDX_RECT_SUB2_OFFS];
                if ((sub1 == -1) && (sub2 == -1)) {
                    if (count == 0) {
                        //System.out.println("Warnning, empty rect: " + i);
                        //return false;
                    }
                } else {
                    if (sub1 >= 0) {
                        if ((sub1 >= nextRect) || (sub1 <= i)) {
                            System.out.println("Error, rect: " + i);
                            return false;
                        }
                    }
                    if (sub2 >= 0) {
                        if ((sub2 >= nextRect) || (sub2 <= i) || (sub2 <= sub1)) {
                            System.out.println("Error, rect: " + i);
                            return false;
                        }
                    }
                }
            }
        }
        return true;
    }

    private void checkObjsCapacity() {
        int l = (objs.length / IDX_OBJ_SIZE);
        if (nextObj > ((l / 10) * 9)) {
            shrink();
            if (nextObj > ((l / 10) * 5)) {
                System.out.println("\nNew size: " + l * IDX_OBJ_SIZE * 2);
                objs = Arrays.copyOf(objs, l * IDX_OBJ_SIZE * 2);
            }
        }
    }

    private void checkRectsCapacity() {
        int l = (rects.length / IDX_RECT_SIZE);
        if (nextRect > ((l / 10) * 9)) {
            rects = Arrays.copyOf(rects, l * IDX_RECT_SIZE * 2);
        }
    }

    public void clear() {
        rects = null;
        objs = null;
        objsCompact = null;
    }
}
